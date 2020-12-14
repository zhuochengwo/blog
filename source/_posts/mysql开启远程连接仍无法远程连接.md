---
title: Mysql开启远程连接仍无法远程连接
tags:
  - MySQL
  - 踩坑
id: '279'
categories:
  - - MySQL
  - - 数据库
  - - 敲代码
date: 2018-09-23 20:30:39
---

一般来说，开发环境为了方便，我们会开启 **root** 的远程连接，开启的方式也很简单，一条授权语句就可以解决。

```mysql
grant all privileges on *.* to 'root'@'%' identified by 'password';

flush privileges;
```

如果这个时候还是无法远程连接，就要考虑 **mysql** 配置文件中是否绑定了 **127.0.0.1**

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

注释掉以下一行，使用#号注释

```vim
# bind-address = 127.0.0.1
```

重启服务

```bash
sudo service mysql restart
```

这是时候就可以通过 **root** 用户远程连接了。 连着碰了两次这个问题，总是忘了 **mysql** 配置项下绑定了 **127.0.0.1** 的问题，这里记录下