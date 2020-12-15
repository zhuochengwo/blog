---
title: Ubuntu日志分割logrotate
tags:
  - Ubuntu
id: '156'
categories:
  - - Ubuntu
  - - 敲代码
date: 2018-06-01 21:38:31
---

对于日志管理，特别是日志文件比较大的时候我们一般都需要转储整理，这里介绍一个非常强大的工具logrotate，我们可以利用它来对日志文件进行统一归档管理。可以根据日志文件的大小，也可以根据其天数来转储。 ubuntu默认已经安装logrotate，logrotate的配置文件是/etc/logrotate.conf，通常不需要对它进行修改。日志文件的轮循设置在独立的配置文件中，它们放在/etc/logrotate.d/目录下。 logrotate常用参数如下：

```vim
daily                     #指定转储周期为每天
weekly                    #指定转储周期为每周；
monthly                   #指定转储周期为每月；
rotate count              #指定日志文件删除之前转储的次数，0指没有备份，5指保留5个备份；
compress                  #通过gzip压缩转储以后的日志；
nocompress                #不需要压缩时，用这个参数；
delaycompress             #延迟压缩，和compress一起使用时，转储的日志文件到下一次转储时才压缩；
nodelaycompress           #覆盖delaycompress选项，转储同时压缩；
copytruncate              #用于还在打开中的日志文件，把当前日志备份并截断；
nocopytruncate            #备份日志文件但是不截断；
create mode owner group   #转储文件，使用指定的文件模式创建新的日志文件；
nocreate                  #不建立新的日志文件；
errors address            #专储时的错误信息发送到指定的Email地址；
ifempty                   #即使是空文件也转储，这个是logrotate的缺省选项；
notifempty                #如果是空文件的话，不转储；
mail address              #把转储的日志文件发送到指定的E-mail地；
nomail                    #转储时不发送日志文件；
olddir directory          #转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统；
noolddir                  #转储后的日志文件和当前日志文件放在同一个目录下；
prerotate/endscript       #在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行；
postrotate/endscript      #在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行；
tabootext [+] list        #让logrotate不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave,v,和~ ；
size size                 #当日志文件到达指定的大小时才转储，Size可以指定bytes(缺省)以及KB(sizek)或者MB(sizem)；
postrotate <s> endscript  #日志轮换过后指定指定的脚本，endscript参数表示结束脚本；
sharedscripts             #共享脚本,下面的postrotate <s> endscript中的脚本只执行一次即可
```

nginx在安装后默认已经启用了logrotate，这里我们参照着nginx的配置对我们shadowsocks的日子做一个归档管理

```bash
cat /etc/logrotate.d/nginx 
```

```vim
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    prerotate
        if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
            run-parts /etc/logrotate.d/httpd-prerotate; \
        fi \
    endscript
    postrotate
        invoke-rc.d nginx rotate >/dev/null 2>&1
    endscript
}
```

我们执行

```bash
sudo cp /etc/logrotate.d/nginx /etc/logrotate.d/shadowsocks
```

为了方便查看，这里不进行压缩，并且去掉了一些脚本操作，加入了dateext以日期作为文件后缀，同时保留30天的日子

```vim
/var/log/shadowsocks/*.log {
    daily
    missingok
    rotate 30
    nocompress
    dateext
    notifempty
    create 0640 chengfeng chengfeng
}
```

修改完毕后使用下边的命令测试配置是否正确

```bash
sudo logrotate -f /etc/logrotate.d/shadowsocks
```

执行后可以看到已经进行了归档。 logrotate需要的**cron**任务应该在安装时就自动创建了，我们不需要做额外的操作

```bash
cat /etc/cron.daily/logrotate 
```

```bash
#!/bin/sh

# Clean non existent log file entries from status file
cd /var/lib/logrotate
test -e status  touch status
head -1 status > status.clean
sed 's/"//g' status  while read logfile date
do
    [ -e "$logfile" ] && echo "\"$logfile\" $date"
done >> status.clean
mv status.clean status

test -x /usr/sbin/logrotate  exit 0
/usr/sbin/logrotate /etc/logrotate.conf
```

