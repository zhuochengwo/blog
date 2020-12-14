---
title: RabbitMq的安装使用
tags: []
id: '271'
categories:
  - - 敲代码
---

前边巴拉巴拉巴拉 1、Signing key

wget -O - 'https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc'  sudo apt-key add -

2、添加源 在 /etc/apt/sources.list.d/ 下新增bintray.erlang.list文件 写入

deb https://dl.bintray.com/rabbitmq/debian $distribution $component

其中： $distribution 指发布版本名，如ubuntu 18.04 是bionic，以下是一些参考  

> bionic for Ubuntu 18.04 xenial for Ubuntu 16.04 stretch for Debian Stretch jessie for Debian Jessie

$component 是[Erlang/OTP Version](http://Erlang/OTP Repositories) 如果是ubuntu18.04的话可以使用如下配置

deb http://dl.bintray.com/rabbitmq/debian bionic erlang

3、安装erlang

sudo apt-get update
sudo apt-get install erlang

可以通过erl查看erlang的版本 4、添加rabbitmq-server signning key

wget -O - 'https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc'  sudo apt-key add -

5、添加rabbitmq-server的源并更新（替换其中的$distribution）

deb https://dl.bintray.com/rabbitmq/debian $distribution main

在 /etc/apt/sources.list.d/ 下新增bintray.erlang.list文件 6、安装rabbitmq-server

sudo apt-get update
sudo apt-get install rabbitmq-server

7、rabbitmq安装完后默认启动 rabbitmqctl status 查看服务状态

sudo rabbitmqctl status

8、安装web监控插件

sudo rabbitmq-plugins enable rabbitmq\_management

可以通过 http://localhost:15672/ 查看监控，默认用户名密码都是guest ![](https://chengfeng.site/src/wp-content/uploads/2018/09/20180918095518-300x117.png) 暂时告一段落。