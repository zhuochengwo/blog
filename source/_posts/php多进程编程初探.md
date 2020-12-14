---
title: php多进程编程初探
tags:
  - PHP
  - php多进程
id: '130'
categories:
  - - PHP
  - - 敲代码
date: 2018-05-06 16:21:37
---

php本身并不支持多进程和多线程，但基于 Linux 的 PHP 扩展 PCNTL 却可以提供多进程编程，所以windows环境是不支持的。 使用[pcntl\_fork](http://php.net/manual/zh/function.pcntl-fork.php)来实现多进程编程

> pcntl\_fork() 函数创建一个子进程，这个子进程仅有 PID（进程号）和 PPID（父进程号）与其父进程不同，其他的都相同。

> 返回值：成功时，在父进程执行线程内返回产生的子进程的 PID，在子进程执行线程内返回 0。失败时，在父进程上下文返回 -1，不会创建子进程，并且会引发一个 PHP 错误。

```php
<?php
// 创建子进程
$pid = pcntl_fork();
if($pid == -1){
    // 返回值为-1，创建失败
    die('could not fork');
}elseif($pid){
    // 返回值大于0，是父进程
    echo "parent \t", date("H:i:s", time()), "\n";
}else{
    // 返回值等于0，是子进程
    echo "child \t", date("H:i:s", time()), "\n";
}
?>
```

如果通过 pcntl\_fork 创建子进程成功的话，系统就有了2个进程，一个为父进程，一个为子进程，子进程的 id 号为 $pid。 为什么一个 if 和 else 互斥的代码中，都输出了结果？ 这恰恰说明了有两个进程在执行后面的代码。fork出来的子进程从$pid = pcntl\_fork()处开始执行，在子进程中执行pcntl\_fork()返回的是0，父进程的变量 $pid 值大于 0，所以执行 elseif 中的代码，子进程的变量 $pid 值等于 0，就会执行 else 中的代码。 至于谁先谁后的问题，这得要看系统资源的分配了。 创建多个进程

```php
<?php
// 创建子进程
for ($i = 0; $i < 2; $i++) {
 $pid = pcntl_fork();
 if ($pid == -1) {
 // 返回值为-1，创建失败
 die('could not fork');
 } elseif ($pid) {
 // 返回值大于0，是父进程,getmypip()能获取当前进程pid
 echo "parent : " . getmypid() . " " . date("H:i:s", time()) . "\n";
 } else {
 // 返回值等于0，是子进程
 echo "child : " . getmypid() . " " . date("H:i:s", time()) . "\n";
 }
}
sleep(60);
?>
```

但看代码想当然应该是创建了两个子进程，但结果呢?

```vim
parent : 22728 13:30:50
parent : 22728 13:30:50
child : 22730 13:30:50
child : 22729 13:30:50
parent : 22729 13:30:50
child : 22731 13:30:50
```

可以看到一共有4个进程，其中22729既是父进程又是子进程。 执行了sleep(60)方便我们使用ps auxf grep php查看具体进程信息

```bash
chengfe+ 22728 0.0 0.2 149588 17992 pts/0 S+ 13:30 0:00   \_ php test.php
chengfe+ 22729 0.0 0.1 149588 8012 pts/0 S+ 13:30 0:00   \_ php test.php
chengfe+ 22731 0.0 0.0 149588 5412 pts/0 S+ 13:30 0:00    \_ php test.php
chengfe+ 22730 0.0 0.0 149588 5412 pts/0 S+ 13:30 0:00   \_ php test.php
```

这样能更清晰的看到，22728是一开始的父进程，其创建了22729和22730两个子进程，而子进程22729又创建了22731这个子进程。 为什么22729会创建自己的子进程呢？而同样是22728的子进程的22730却没有创建自己的子进程？ 当 fork 出来子进程后，它就与父进程完成独立了，并保持着之前的执行状态。从上面的结果可以推断出，当（$i == 0）第一次循环时，父进程为 22728，通过 fork 它得到了一个子进程 22729，然后分别执行各自的代码；父进程22728继续执行第二次循环并创建了子进程22730，同时它的子进程22729也会继续执行第二次循环并创建自己的子进程22731，因为22729它的执行状态和父进程22728是一样的。而22730和22731均是在第二次循环的时候创建的，创建完后后续执行就退出循环不会再继续创建新的子进程了。 若不想父进程创建的子进程再创建新的子进程，只需在判断子进程的分支break掉循环即可。

```php
for ($i = 0; $i < 2; $i++) {
    $pid = pcntl_fork();
    if ($pid == -1) {
        die('could not fork');
    } elseif ($pid) {
        echo "parent : " . getmypid() . "  " . date("H:i:s", time()) . "\n";
    } else {
        echo "child : " . getmypid() . "   " . date("H:i:s", time()) . "\n";
        break;
    }
}
sleep(60);
```

输出如下：

```vim
parent : 22858 13:46:04
child : 22859 13:46:04
parent : 22858 13:46:04
child : 22860 13:46:04

chengfe+ 22858 0.0 0.2 149588 17768 pts/0 S+ 13:46 0:00   \_ php test.php
chengfe+ 22859 0.0 0.1 149588 7928 pts/0 S+ 13:46 0:00   \_ php test.php
chengfe+ 22860 0.0 0.0 149588 5412 pts/0 S+ 13:46 0:00   \_ php test.php
```

这样结果就跟预期一样了。