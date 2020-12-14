---
title: 基于RabbitMQ和Swoole实现异步任务消费系统
tags: []
id: '215'
categories:
  - - 敲代码
---

一个异步任务系统，最重要的部分就是消费端，本篇文章，主要介绍基于RabbitMQ和Swoole实现的异步任务消费系统。 这里为了方便演示，模拟一个场景：

1.  用户创建订单并支付后系统通过事件生产者发送消息到RabbitMQ的orders exchange
2.  RabbitMQ的orders 这个exchange绑定了两个queen，一个是order\_notify模拟系统给用户发送推送消息（短信，push等），另一个queen是order\_delivery模拟系统处理发货任务。为了方便，这里用的是RabbitMQ的[Publish/Subscribe](https://www.rabbitmq.com/tutorials/tutorial-three-php.html)模式。
3.  两个queen通过两个swoole\_server充当消费者，每个swoole\_server都有数个task\_worker实际消费消息。

## 准备

*   首先确认安装好rabbitmq-server以及php和Swoole扩展
*   提前创建exchange和queen并绑定，这里exchange名用orders，两个queen分别是order\_notify,order\_delivery。可以通过rabbitmqctl也可以通过web控制台操作
*   如果想用web控制台图形化操作的话，需要启用rabbitmq\_management，通过命令
    
    rabbitmq-plugins enable rabbitmq\_management
    
    启用，就可以通过http://localhost:15672访问控制台，默认用户名密码均为guest

## **事件生产者**

这里为了简化系统，侧重于消费端的实现，生产者简单模拟用户创建订单。

这里有代码

## 消费者

基于swoole的消费者