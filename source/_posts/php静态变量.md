---
title: PHP静态变量
tags:
  - PHP
id: '142'
categories:
  - - PHP
  - - 敲代码
date: 2018-05-13 14:22:40
---

PHP一般用作web开发，处理http请求，例如nginx + php-fm。FPM 是一个 PHP 进程管理器，包含 master 进程和 worker 进程两种进程：master 进程只有一个，负责监听端口，接收来自 Web Server (nginx)的请求，而 worker 进程则一般有多个 (具体数量根据实际需要配置)，每个进程内部都嵌入了一个 PHP 解释器，是 PHP 代码真正执行的地方。不同的请求是在不同的php-fpm进程中解析执行的，每个进程(系统进行资源分配和调度的基本单位)是独享内存的互不干扰。php-fpm每次都会初始化，静态变量即在此刻写入内存，这种情况下，虽然程序代码中使用的是static，但是这个static只对单个请求生效，请求之间是互相隔离不会造成干扰的。 但随着swoole出现，php的编程方式也带来了一些变化，其中之一就是有了通俗意义上的static。 我们来试着测试下基于swoole的http server

```php
<?php
//全局变量
$show = 'hello world';

class Test
{
    public static $hello = 'hello test';

}

$http = new swoole_http_server("127.0.0.1", 9501);

$http->on("start", function ($server) {
    echo "Swoole http server is started at http://127.0.0.1:9501\n";
});

$http->on("request", function ($request, $response) {
    $response->header("Content-Type", "text/plain");
    global $show;
    if (!empty($request->get)) {
        if (!empty($request->get['global'])) {
            echo "设置全局变量值为{$request->get['global']}\n";
            $show = $request->get['global'];
        }
        if (!empty($request->get['static'])) {
            echo "设置类静态变量值为{$request->get['static']}\n";
            Test::$hello = $request->get['static'];
        }
    }
    $info = "类静态变量:" . Test::$hello . " #### " . "全局变量:$show\n\n\n";
    echo $info;
    $response->end($info);
});

$http->start();
```

这段代码测试全局变量和类静态变量是否在每次请求中会被保留下来。 使用php server.php直接启动http server，方便在控制台查看输出的内容，使用curl传入不同的get参数来进行测试。 curl调用输出内容如下：

```bash
curl "127.0.0.1:9501"
类静态变量:hello test #### 全局变量:hello world


curl "127.0.0.1:9501?global=aaaaaa&static=11111" 
类静态变量:11111 #### 全局变量:aaaaaa


curl "127.0.0.1:9501"                           
类静态变量:11111 #### 全局变量:aaaaaa

```

server输入内容如下：

```bash
test php server.php 
Swoole http server is started at http://127.0.0.1:9501

类静态变量:hello test #### 全局变量:hello world


设置全局变量值为aaaaaa
设置类静态变量值为11111
类静态变量:11111 #### 全局变量:aaaaaa


类静态变量:11111 #### 全局变量:aaaaaa

```

第一次请求，没有get参数，输出全局变量$show和test类静态变量$hello的默认值。 第二次请求，传入get参数，设置全局变量$show为aaaaaa，设置test类静态变量$hello的值为11111，可以看到打印出的内容发生了变化，说明变量被修改了。 第三次请求，没有get参数，输出结果同第二次请求相同，说明全局变量和类静态变量的值不会在请求结束后释放，能一直保存在内容中。 通过这个简单的例子可以看到，swoole这种常住内存的方式是能提供通俗意义上静态变量的实现，不同于php-fpm的处理方式的。在编程中使用swoole是需要特别注意这些区别，避免在使用静态变量是发生出乎意料的bug。