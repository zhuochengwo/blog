---
title: Centos安装docker compose重启导致root无法ssh登录
tags:
  - Docker
  - 踩坑
id: '265'
categories:
  - - 敲代码
date: 2018-09-14 13:00:47
---

近日，有网友反馈在虚拟机的 **centos** 上安装 **docker compose** 重启虚拟机后，出现**root**用户无法 **ssh** 登录的情况。 我一般很少直接用 **root** 用户登录，所以也没注意过这个问题，下了个 **centos 7.5** 的版本用虚拟机选择最小安装，随后安装 **docker** 及 **docker compose** ，重启电脑，期待复现问题。没想到还真出现了，确实有 **root** 无法登录的情况，回看了下整个操作，基本确定也就是 **docker compose** 导致的。 普通用户还是可以正常登陆的，于是用 **xshell** 连接虚拟机登陆上普通用户，看了下，**docker** 的服务没有启动，因为安装后没有添加开机自启动，抱着试一试的态度，启动了 **docker** 服务

```bash
sudo service docker start
```

而后再次尝试 **xshell** 使用 **root** 登陆，发现能正常登陆了。 在 **/var/log/secure** 日志中找到一条

> pam\_succeed\_if(sshd:auth): requirement "uid >= 1000" not met by user "root"

检查了下  **/etc/pam.d/**  下的配置，也没能发现什么异常，抱着一探究竟的心态，重新安装虚拟机尝试定位问题，很遗憾的是，没能再出现这个问题。 **目前有点无解了，记录下，改天再试试。**