---
title: Mysql主从配置
tags:
  - MySQL
id: '105'
categories:
  - - MySQL
date: 2018-04-14 11:31:39
---

## 一、配置master（主服务器）

```bash
vim /etc/my.cnf
```

\***\[必须\]启用二进制日志**

```vim
og-bin=mysql-bin
```

**\*\[必须\]服务器唯一ID，默认是1，一般取IP最后一段** 

```vim
server-id=151
```

## 二、配置slave（从服务器）

```bash
vim /etc/my.cnf
```

\***\[可选\]启用二进制日志**

```vim
log-bin=mysql-bin
```

\***\[必须\]服务器唯一ID，默认是1，一般取IP最后一段**

```vim
server-id=152
```

## 三、在主服务器上创建备份专用帐户

```bash
mysql -uroot -ppassword -e "GRANT REPLICATION SLAVE,RELOAD,SUPER ON *.* TO 'backup'@'192.168.0.154' IDENTIFIED BY '123456';"
```

\***查询master(主服务器)的状态**

```bash
mysql -uroot -ppassword -e "show master status;"
```

## 四、配置Slave启动主从复制

\***注意不要断开，154数字前后无单引号。** \***启动从服务器复制功能**

```bash
mysql -uroot -ppassword -e "change master to master_host='192.168.0.151',master_user='backup',master_password='123456',master_log_file='mysql-bin.000001',master_log_pos=154; start slave;"
```

*   master\_host=主服务器IP
*   master\_user=在主服务器上创建的备份用户名
*   master\_password=备份用户密码
*   master\_log\_file=查询master(主服务器)的状态得到的File列的值
*   master\_log\_pos=Position列的值
*   start slave：启动从服务器复制功能 检查从服务器复制功能状态

```bash
mysql -uroot -ppassword -e "show slave status\G;"
```