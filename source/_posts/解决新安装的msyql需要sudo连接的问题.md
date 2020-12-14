---
title: 解决msyql需要sudo连接的问题
tags:
  - MySQL
  - 踩坑
id: '167'
categories:
  - - MySQL
  - - 数据库
  - - 敲代码
date: 2018-06-02 15:04:53
---

在ubuntu上使用安装完mysql后，但是首次无密码连接却被拒，只能sudo连接。 在Stack Overflow上找到了相关说明，原因是mysql使用了auth\_socket plugin进行认证。

> If you install 5.7 and don’t provide a password to the root user, it will use the auth\_socket plugin. That plugin doesn’t care and doesn’t need a password. It just checks if the user is connecting using a UNIX socket and then compares the username.

解决方法是使用msyql自带密码认证功能

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

退出后就可以使用上边命令设置的密码登录了，不再需要sudo。