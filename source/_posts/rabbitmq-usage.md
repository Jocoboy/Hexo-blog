---
title: RabbitMQ核心概念及应用场景
date: 2024-08-17 08:20:47
categories:
- Middleware
tags:
- RabbitMQ
---

RabbitMQ基本概念、安装配置、使用场景、消息模型，以及消息持久化、签收机制、延迟队列的实现。

<!--more-->

## 前言

RabbitMQ是基于AMQP(Advanced Message Queue Protocol)协议实现的消息队列，是一种应用程序之间的通信方法，在分布式系统开发中应用广泛。

## 基本概念

{% asset_img rabbitmq_architecture.png RabbitMQ工作原理 %}

### Broker

Broker是消息队列进程，包含Exchange、Queue两个部分。

#### Exchange

Exchange指消费队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过滤。

常用类型有direct、topic、fanout、headers四种。

#### Queue

Queue指消息队列，消息打达队列后转发给指定的消费方。

### Producer

Producer指消息生产者，生产者发布消息的过程如下：

- 生产者和broker建立Connection，并开启一个channel

- 生产者声明一个交换机并设置相关属性(交换机类型、是否持久化)

- 生产者声明一个队列并设置相关属性(是否排他、是否持久化、是否自动删除)

- 生产者通过routing key将交换机和队列绑定

- 生产者通过channel发送给broker，由交换机根据接收到的routing key匹配队列

- 如果找到匹配的队列则存入，否则丢弃或回传

### Consumer

Consumer指消息消费者，消费者接受消息的过程如下：

- 消费者和broker建立Connection，并开启一个channel

- 消费者监听指定的队列，根据需要设置回调函数

- 当有消息到达队列时broker将消息推送给消费者

- 消费者确认接受到消息

- broker删除队列中已经确认的消息

## 使用场景

RabbitMQ通常有如下应用场景：

- 高并发场景消除峰值，让并发请求在MQ中排队
- 大数据处理，将数据放入MQ，多开几个消费者处理(如日志收集)
- 服务异步和解耦，使用MQ进行异步通信后，服务之间没有直接的调用关系，生产方通过MQ与消费方通信，将应用程序进行解耦
- FIFO排序，保证数据按顺序消费

## 安装配置

Windows系统中RabbitMQ可通过安装包下载(需要Erlang环境)，安装目录下启动图形化界面并重启服务

`rabbitmq-plugins enable rabbitmq_management`

`rabbitmq-server stop`

`rabbitmq-server start`

也可通过Dokcer下载镜像

`docker pull rabbitmq:3-management`

下载完成后启动容器(5672是程序连接的端口，15672是可视化界面接口)

`docker run -id --name=rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management`

## 消息模型与交换机类型


RabbitMQ包含以下5种消息模型

- Hello World简单模型，只需一个生产者、一个队列、一个消费者

- Work Queue工作队列模型，多个消费者绑定到一个队列，共同消费队列中的消息，同一个消息只会被一个消费者消费。可使用prefetch防止某个消费者消费能力偏弱导致后续的消息阻塞

- Publish/Subscribe发布订阅模型(type=fanout)，允许一个消息向多个消费者投递

- Routing路由模型(type=direct)，不同的消息可被不同的队列消费，通过一个routing key来收发消息

- Topic通配符模型(type=topic)，一种特殊的路由模式，在绑定队列时routing key可以使用通配符

注: headers类型的路由不是用routing Key进行路由匹配，而是在匹配请求头中所带的键值进行路由

## 消息持久化

MQ消息在内存中进行读写，如果MQ宕机那么消息就要丢失的风险，我们需要可以通过交换机持久化、队列持久化等来防止消息丢失。

## 签收机制

RabbitMQ包含手动签收和自动签收2钟模式。

自动签收指MQ把消息投递给消费者后，消息默认被签收，MQ就会直接把消息删除掉。这种模式可能会导致消息丢失分享。

手动签收模指MQ不会自动签收消息，而是把消息推送给消费者后，等到消费者自己去签收消息后，再删除队列中的消息，这种模式可以防止消息丢失。

## 延迟队列和死信消息

RabbitMQ可以给队列设置过期时间，也可以单独给每个消息设置过期时间，如果到了过期时间消息没被消费该消息就会标记为死信消息。根据这一特点，我们可以准备一个队列来接收死信交换机中的死信消息，然后准备一个消费者来消费该队列中的消息，这就是延迟队列。

## 参考文档

- [RabbitMQ官方文档](https://www.rabbitmq.com/docs)