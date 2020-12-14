---
title: 独立使用Eloquent之sql日志及重连机制
tags:
  - Eloquent
  - MySQL
  - PHP
id: '285'
categories:
  - - PHP
  - - 敲代码
date: 2018-09-24 00:40:45
---

## 综述

最近写的项目是一个后台常驻进程的项目，没有引入框架，操作数据库这一块为了方便引入了 [Eloquent](https://github.com/illuminate/database) 。 **Laravel** 是目前 **PHP** 语言最活跃、最优雅的框架，作为 **Laravel** 框架核心组件之一的 **Eloquent ORM** 使人印象深刻。得益于 **Laravel** 组件化的设计 ([Laravel组件库](https://github.com/illuminate) ) ，使得我们能在 **Laravel** 之外仍能非常方便的单独使用，这一点必须为 **Laravel** 设计者们点赞。 使用上也非常简单，直接通过 **composer** 安装即可。

```bash
composer require illuminate/database
```

对于数据库及密码这一类隐私类的配置，我们也采用 **.env** 文件进行配置，并对其忽略，避免提交到版本库中。对于 **.env** 文件的读取，可以使用[phpdotenv](https://github.com/vlucas/phpdotenv)这个库，使用上只需通过

```php
//这里的ROOT_PATH是定义的项目根目录，也是.env文件所在目录
$dotenv = new Dotenv(ROOT_PATH);
$dotenv->load();
```

加载即可，这样可以通过 **php** 系统函数 **getenv** 或者超全局变量 **$\_ENV** 和  **$\_SERVER** 中取到我们在 **.env** 中的配置了。

```php
use Illuminate\Database\Capsule\Manager as Capsule;

$capsule = new Capsule;
//配置统一放置在了.env中
$capsule->addConnection([
    'driver'    => 'mysql',
    'host'      => getenv('db_host'),
    'database'  => getenv('db_database'),
    'username'  => getenv('db_username'),
    'password'  => getenv('db_password'),
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
]);
//设置全局，这样就能在项目中全局使用了
$capsule->setAsGlobal();
//启动
$capsule->bootEloquent();
```

几行代码就搞定了，接下来就像在**Laravel**中一样方便的操作数据库了。

### 1、sql日志

开发中，免不了需要查看 **sql** 日志，**Eloquent** 组件提供了注册 **dispatcher** 的方法，这样我们就能通过捕获 **Eloquent** 的事件来打印日志了。这里主要参考了[这篇博客](http://xieye.iteye.com/blog/2387809)，此博主关于独立使用 **Eloquent** 有一系列的文章，值得一看。

#### 1.1、创建dispatcher

```php
namespace app\dispatcher;
use \Illuminate\Contracts\Events\Dispatcher;

class SqlDispatcher implements Dispatcher
{
    public function dispatch($event, $payload = [], $halt = false)
    {
        //读取QueryExecuted事件中的sql内容并打印
        if ($event instanceof \Illuminate\Database\Events\QueryExecuted) {
            $sql = $event->sql;
            if ($event->bindings) {
                foreach ($event->bindings as $v) {
                    $sql = preg_replace('/\\?/', "'" . addslashes($v) . "'", $sql, 1);
                }
            }
            echo $sql . PHP_EOL;
        }
    }

    public function listen($events, $listener){}
    public function hasListeners($eventName){}
    public function subscribe($subscriber){}
    public function until($event, $payload = []){}
    public function push($event, $payload = []){}
    public function flush($event){}
    public function forget($event){}
    public function forgetPushed(){}
}
```

#### 1.2、注册dispatcher

在之前 **MySQL**初始化 **Eloquent** 的地方添加一行代码即可

```php
$capsule->setAsGlobal();
$capsule->bootEloquent();
//注册刚创建的dispatcher
$capsule->setEventDispatcher(new SqlDispatcher());
```

因为暂时只需要 **sql** 日志，这个 **dispatcher** 写的非常简单，其实它可以将 **Eloquent** 事件按照添加的 **listener** 继续分发。

### 2、Eloquent重连机制

因为这是一个常驻进程的项目，那么就必须考虑 **MySQL**重连问题，一方面，**MySQL**有连接超时的问题（通过 **wait\_timeout** 设置），另一方面要考虑 **MySQL** 重启的问题，这两者任意一个都可能导致 **MySQL server has gone away** 的问题。 解决  **MySQL server has gone away** 的问题有好几种解决方案，目前来说最佳方案就是在执行 **mysql\_query** 时检查执行结果，如果发现断开则自动重连，并重新执行 **sql** ，**Eloquent** 就是使用的这种方案。 查看 **Eloquent** 底层源码，可以在**vendor/illuminate/database/Connection.php**中看到有 **tryAgainIfCausedByLostConnection** 方法来处理这个问题。 这里主要是验证下这个重试机制。

#### 2.1、设置mysql超时时间

#### 2.2、测试mysql超时情况下的重试问题

将 **MySQL** 的超时时间设置成 **30s**

```sql
set global wait_timeout=30;
```

```php
while (true) {
    $user = User::find(1);
    echo date('Y-m-d H:i:s') . PHP_EOL;
    echo "UserId:" . $user->id . PHP_EOL;
    echo "\n------------------------------------------\n";
    sleep(40);
}
```

![](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180923233957.png) 通过执行代码输出的内容可以清晰看到的看到一个 **php warning** ，出现了**MySQL server has gone away**的问题，而 **Eloquent** 捕获了这个错误，自动重连了并重新执行了查询语句获取了结果。其中打印出来的 **sql** 就是之前添加的 **SqlDispatcher** 输出的。 通过 **MySQL**的 **show processlist** 查看连接情况，也可以看到重连结果。

```sql
show processlist;
```

![]( https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180923233918.png) 可以发现在超过 **30s** 的时候 **MySQL**关闭了连接，而后 **Eloquent** 自动重连了，新的连接重新开始计时。

#### 2.3、测试mysql重启情况下的重试问题

从前一个例子可以得知，**MySQL**重新的情况下应该也是可以自动重连的，两种情况并没有本质区别。抱着严谨的态度，还是来测试下重启 **MySQL**的情况下。 简单的将之前的测试脚本中 **sleep** 时间改成 **10** ，然后通过重启 **MySQL** 进行测试

```bash
sudo service mysql restart
```

![](    https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180924000024.png) 查看结果后发现确实是同样的结果，证明 **Eloquent** 的重试机制还是可靠的。

# 总结

脱离**Laravel**独立使用 **Eloquent** 还是非常方便的，几乎没有成本。其自带的重试机制对于常驻进程的应用也多了一层可靠性保护。目前来说有点问题的是 **Eloquent** 的实现对如 **PhpStorm** 等IDE不够友好，代码提示缺失，文档上只能依赖于 **Laravel** 的文档，使用中并不是特别顺手。