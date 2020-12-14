---
title: 解决Laravel修改phpredis导致travis-ci失败的问题
tags:
  - laravel
  - 踩坑
id: '504'
categories:
  - - Travis-ci
  - - 敲代码
date: 2019-06-26 14:56:09
---

`Laravel`默认使用的是[predis](https://github.com/nrk/predis)这个库来操作`Redis`的，想把它改成`PHP`扩展的`phpredis`，在`config/database.php`中将`client`值从`predis`改成`phpredis`即可。

```php
'redis' => [
        'client' => 'phpredis',
]
```

本地测试一切正常，但是在推送的`github`后`travis-ci`却失败了，失败是在`composer install`后触发`dump-autoload`是报了一个 **_Call to undefined method Illuminate\\Support\\Facades\\Redis::connect()_** 原因是`travis-ci`环境没有安装`phpredis`这个扩展。 解决也很简单，修改`.travis.yml`文件，在`install`阶段通过`pecl`来安装`phpredis`即可。需要注意的是`pecl`安装`phpredis`时有两个参数是通过交互界面输入的，我们需要通过管道来输入两个回车让`CI`继续下去。

```vim
install:
  - print "\n\n"  pecl install redis
  - composer install
```

至此，`CI`就可以成功的继续下去了。 对于`travis-ci`失败，我们可以通过`travis-ci`提供的`debug`模式调试。 在`jobs`详情界面，在右侧`Restart job`下方有一个`Debug job`的按钮，点击后开启debug任务，在下方`Job log`中会输出一个可以`ssh`连接的地址

```vim
Preparing debug sessions.
Use the following SSH command to access the interactive debugging environment:
ssh xxxx@xxxx
This build is running in quiet mode. No session output will be displayed.
```

通过ssh连接到虚拟机进行调试，解决完问题后，将相关命令补充到`.travis.yml`。