---
title: Python多线程编程
tags:
  - Python
id: '26'
categories:
  - - Python
date: 2018-02-02 16:11:50
---

尝试着写了下`python`的多线程，发现还是挺简单的，这里用代码简单记录下： 开10个线程，让其中一个执行失败

```python
import threading

all_config = []

def do(n):
    i = 10000000
    print('thread %s is running...' % threading.current_thread().name)
    j = 0
    while i > 0:
        i = i - 1
        j = 2 ** 8
    if n == 4:
        print('not ok')
        exit(0)
    print('thread %s is run ok!!!' % threading.current_thread().name)
    all_config.append({'a': n})


def do_run():
    threads = []
    for i in range(1, 10):
        t = threading.Thread(target=do, args=(i,))
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join()


print('thread %s is running...' % threading.current_thread().name)
do_run()
print(all_config)

```

但是这种方式有一个问题，比如，我在n==4的子线程中模拟发生了异常导致线程退出，但是，在主线程中并不能catch到子线程的异常，并做出相应的处理，如果想捕获这个异常，可能这样来写

```python
import threading

all_config = []


class Worker(threading.Thread):
    def __init__(self, n):
        self.n = n
        self.exitcode = 0
        threading.Thread.__init__(self, name='get_config_' + str(n))

    def run(self):
        i = 10000000
        print('thread %s is running...' % threading.current_thread().name)
        j = 0
        while i > 0:
            i = i - 1
            j = 2 ** 8
        if self.n == 4:
            self.exitcode = 1
            exit(0)
        print('thread %s is run ok!!!' % threading.current_thread().name)
        all_config.append({'a': self.n})


def run():
    threads = []
    for i in range(1, 10):
        t = Worker(i)
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join(.1)
    while len(threads) > 0:
        for k, thread in enumerate(threads):
            if thread.exitcode != 0:
                print('child exception happened')
            if thread.isAlive():
                continue
            threads.pop(k)
            print('thread %s is running...' % 

threading.current_thread().name)
run()
print(all_config)
```

这样就可以在主线程中通过判断exitcode的值来处理线程的异常 不过这种写法并不好，还在摸索中，等python玩的更熟了以后再补充