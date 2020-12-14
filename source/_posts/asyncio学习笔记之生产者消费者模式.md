---
title: asyncio学习笔记之生产者消费者模式
tags:
  - Python
id: '63'
categories:
  - - Python
  - - 敲代码
date: 2018-02-12 18:07:29
---

在写一个小爬虫的时候想用下python的asyncio，用协程的方式来实现。 看官方文档 [asyncio](https://docs.python.org/3/library/asyncio-queue.html)，好像很简单的样子，于是哐嘁哐嘁动手就搞起来了，结果还是踩了不少坑。文档看起来简单，等踩到坑的时候才发现文档根本帮不上忙了。各种找资料看别人的实现，还是花了点时间的。 准备实现的功能也很简单，先给到一个links列表表示所有需要爬取的页面，通过爬虫将数据爬下来后丢到一个queue里，用此作为生产者，另外有消费者去处理这个queue里的数据（比如入库，分析等），场景中生产者比消费者耗时。 最开始，我是参考[这篇文章](https://zhuanlan.zhihu.com/p/32754616)来实现的，结果就踩了个坑，浪费了不少时间。 以下是最开始的代码

```python
class Test(object):

    def __init__(self):
        self.loop = asyncio.get_event_loop()
        self.urls_queue = asyncio.Queue(64)
        self.links = [i for i in range(20)]
        self.results = []

    def run(self):
        try:
            self.loop.run_until_complete(self.handler())
        finally:
            self.loop.stop()
            self.loop.close()
        print(self.results)
        print(len(self.results))

    async def handler(self):

        self.urls_queue.put_nowait((None, None))

        producers = [asyncio.ensure_future(self.__fetch()) for _ in range(4)]
        workers = [asyncio.ensure_future(self.__worker()) for _ in range(10)]

        await self.urls_queue.join()

        for worker in workers:
            worker.cancel()
        for producer in producers:
            producer.cancel()

    async def __worker(self):
        while True:
            config, content = await self.urls_queue.get()
            if not config or not content:
                continue
            await asyncio.sleep(0.5)
            self.results.append(content)
            print('Worker handler:' + str(content))
            self.urls_queue.task_done()

    async def __fetch(self):
        while len(self.links) > 0:
            config = self.links.pop()
            data = await self.__provider(config)
            print('Producer put:' + str(data))
            await self.urls_queue.put((config, data))

    async def __provider(self, config):
        await asyncio.sleep(0.01)
        return config * 10


test = Test()
test.run()
```

这段代码是通过queen.join()来阻塞等待任务执行完毕的，而queen.join()的退出需要queen.task\_done()的通知。上述代码少了一次task\_done()的调用，因为在一开始self.urls\_queue.put\_nowait((None, None))这里的内容被消费后没有task\_done()通知，所以这个任务会之一阻塞。但何时来调用最后一次task\_done确是个问题，因为要保证producer和customer都执行完毕，所以这种方式其实是不太满足我们的需求的，当然可以实现，但会使代码复杂化，其实有更简单的方式实现。 以下是最后满足需求的代码：

```python
import asyncio

results = []


async def consumer(n, q):
    while True:
        item = await q.get()
        print('consumer {}: get item {}'.format(n, item))
        # 在这个程序中 None 是个特殊的值，表示终止信号
        if item is None:
            break
        else:
            await asyncio.sleep(0.1)
            results.append(item * 10)
    print('consumer {}: ending'.format(n))


async def producer(q, links, j):
    print('producer {}: is working'.format(j))
    while len(links) > 0:
        data = links.pop()
        await asyncio.sleep(0.3)
        await q.put(data)
        print('producer {}: added item {} to the queue'.format(j, data))
    print('producer {}: ending'.format(j))


async def main(loop):
    q = asyncio.Queue(maxsize=64)
    links = [i for i in range(20)]

    # 调度生产者、消费者
    producers = [loop.create_task(producer(q, links, j)) for j in range(4)]
    consumers = [loop.create_task(consumer(i, q)) for i in range(2)]

    # 等待所生产者完成
    await asyncio.wait(producers)
    # 生产者完成任务后通知消费者停止，通过发送一个None标记来实现
    for i in range(4):
        await q.put(None)
    # 等待消费者结束
    await asyncio.wait(consumers)
    for c in consumers:
        c.cancel()
    for p in producers:
        p.cancel()


event_loop = asyncio.get_event_loop()
try:
    event_loop.run_until_complete(main(event_loop))
finally:
    event_loop.stop()
    event_loop.close()
print(results)
print(len(results))
```

首先通过asyncio.Queue(maxsize=64)定义一个最大长度64的队列，并初始化一个虚拟的links来表示已经准备好的所有需要爬取的页面链接，创建了4个生产者，2个消费者，因为上文已经说了，在这个场景中生产者爬取数据会比较耗时，相对来说消费者处理会更快，会出现阻塞等待生产的情况。 再看生产者代码，while True执行到links为空表示所有内容已经抓取，asyncio.sleep(0.3)模拟抓取的过程中网络延时等消耗。这里生产者只是简单的将links的数据推到queue。 消费者这边，阻塞的从队列获取数据，简单的乘以10后推到results中表示处理完成，并用asyncio.sleep(0.1)表示处理花的时间。如果从queue中获取出来的内容是约定好的None，则表示生产者已经处理结束了，不会再有新的数据，消费者也应该退出。如果没有这个结束标记，消费者会一直等待，即便queue为空。 在调度这里，等待生产者全部结束后推送结束标记给消费者通知消费者，在等待消费者结束，完成整个过程 执行结果：

```bash
producer 0: is working
producer 1: is working
producer 2: is working
producer 3: is working
producer 0: added item 19 to the queue
producer 1: added item 18 to the queue
producer 2: added item 17 to the queue
producer 3: added item 16 to the queue
consumer 0: get item 19
consumer 1: get item 18
consumer 0: get item 17
consumer 1: get item 16
producer 0: added item 15 to the queue
producer 1: added item 14 to the queue
producer 2: added item 13 to the queue
producer 3: added item 12 to the queue
consumer 0: get item 15
consumer 1: get item 14
consumer 0: get item 13
consumer 1: get item 12
producer 0: added item 11 to the queue
producer 1: added item 10 to the queue
producer 2: added item 9 to the queue
producer 3: added item 8 to the queue
consumer 0: get item 11
consumer 1: get item 10
consumer 0: get item 9
consumer 1: get item 8
producer 0: added item 7 to the queue
producer 1: added item 6 to the queue
producer 2: added item 5 to the queue
producer 3: added item 4 to the queue
consumer 0: get item 7
consumer 1: get item 6
consumer 0: get item 5
consumer 1: get item 4
producer 0: added item 3 to the queue
producer 0: ending
producer 1: added item 2 to the queue
producer 1: ending
producer 2: added item 1 to the queue
producer 2: ending
producer 3: added item 0 to the queue
producer 3: ending
consumer 0: get item 3
consumer 1: get item 2
consumer 0: get item 1
consumer 1: get item 0
consumer 0: get item None
consumer 0: ending
consumer 1: get item None
consumer 1: ending
[190, 180, 170, 160, 150, 140, 130, 120, 110, 100, 90, 80, 70, 60, 50, 40, 30, 20, 10, 0]
20
```

后续补充，通过另外一个协程来生成links模拟另一个爬虫爬取所有待处理链接。