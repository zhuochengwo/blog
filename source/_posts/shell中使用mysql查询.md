---
title: shell中使用mysql查询并实现博客数据库自动备份
tags:
  - MySQL
  - Shell
id: '249'
categories:
  - - MySQL
  - - Shell
  - - 敲代码
date: 2018-08-28 16:34:58
---

### 需求

之前博客的数据库一直都是手动备份的，一直想写个自动任务一直没写。想法是在有新增文章的情况下，mysqldump出数据库，按日期保存至项目中并自动push到gitlab。要实现也比较简单，查下当前数据库最新文章，对比下时间戳就行了，顺便练习下shell就用shell写了。

### 走起

#### 查询sql

通过查询最近一篇文章的修改时间与当前时间对比来确认是否需要备份操作

```sql
SELECT UNIX_TIMESTAMP(post_modified) FROM posts ORDER BY ID DESC LIMIT 1
```

因为连接数据库涉及到账号密码，而代码是在版本库中，所以一开始的想法是在本地写一个文件保存账户密码，通过shell读取到里边的账户密码通过mysql的-e参数直接执行查询语句，并将结果赋值给变量

```bash
mysql -uxxx -pxxx -Dxxx -e "sql"
```

实际使用的时候发现点问题，mysql在5.6版本以后，出于安全方面的考虑，在终端通过明文密码连接数据库操作会有一个warning，而且找了下相关文档还不能屏蔽掉。于是，换了一种方式，在[Stack Overflow](https://stackoverflow.com/questions/20751352/suppress-warning-messages-using-mysql-from-within-terminal-but-password-written)上找到一种方法，可以简单的解决这个问题。可以通过mysql\_config\_editor提前设置好账号密码，通过mysql的--login-path参数读取即可。

#### mysql\_config\_ediror设置

```bash
mysql_config_editor set --login-path=blog --host=localhost --user=xxx --password
```

通过mysql\_config\_editor print可以查看到设置的内容

```vim
➜  ~ mysql_config_editor print --login-path=blog
[blog]
user = xxx
password = *****
host = localhost
```

这样在shell中只需要通过

```bash
mysql --login-path=blog -Dblog -A -N -e "sql"
```

读取查询结果，对比即可。 在执行sql之前，可以在shell中加入检查mysql\_config\_editor设置，避免后续错误

### 完整代码如下

```bash
#!/bin/bash
#获取数据库连接配置
path=`dirname $0`
cd $path
timestamp=`date +%s`
conf=`mysql_config_editor print --login-path=blog`
filename="blog_`date +%F`.sql"
logfile='run.log'


if [ ! "$conf" ]
then
    echo "set mysql_config_editor at first:[mysql_config_editor set --login-path=blog --host=localhost --user=xxx --password]"
    exit 1
fi
check_sql="SELECT UNIX_TIMESTAMP(post_modified) FROM posts ORDER BY ID DESC LIMIT 1"
res=`mysql --login-path=blog -Dblog -A -N -e "$check_sql"`
log="[`date +%F` `date +%T`]"
#有新文章的情况下会执行mysqldump并push到远程仓库
if [ $res -gt `expr $timestamp - 86400` ]
then
    mysqldump --login-path=blog blog > $filename
    git add .
    git commit -m "backup blog automatic -- $log"
    git push origin master
    log="$log blog backup done"
else
    log="$log without new article"
fi
echo $log >> $logfile
```

再通过`crontab`添加定时任务每天执行一次就好了

```vim
#每天凌晨3点备份博客数据库，/path/backup.sh是脚本路径
0 3 * * * bash /path/backup.sh >> /tmp/blog_backup.log 2>&1
```

这样只要在24小时内有文章的修改就会在每天凌晨3点触发备份了