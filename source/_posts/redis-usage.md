---
title: Redis数据库常用CLI命令
date: 2024-08-14 10:58:38
categories:
- Database
tags:
- Redis
- NoSQL
---

Redis数据库的使用场景、使用方法、重要概念，以及一些常用的CLI命令。

<!--more-->

## 前言

Redis(Remote Dictionary Server)是一个高性能的(key/value)分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，常用于数据缓存、充当消息队列等。Redis不仅仅支持简单的key-value类型的数据(如string)，同时还提供list，set，sorted set，hash等数据结构的存储。

## 使用方法

Redis在Mac/Linux系统下可通过命令行直接安装，而在Windows系统下需要通过WSL安装一个Linux系统来安装Redis，或者通过Docker下载Redis镜像通过镜像运行Redis，也可以使用传统EXE安装包安装(不推荐)。Redis可通过CLI、API、GUI(如RedisInsight)三种方式使用。

### 启动Redis服务

启动Redis

`redis-server`

报错
> Could not create server TCP listening socket *:6379: bind: 在一个非套接字上尝试了一个操作 。

Redis安装目录下，依次输入

`redis-cli.exe`

`shutdown`

`exit`

`redis-server.exe redis.windows.conf`

### CLI常用命令

#### 数据结构

##### key-value

设置键值对

`SET [key] [value]`

获取键值

`GET [key]`

删除键值对

`DELETE [key]`

判断键值对是否存在

`EXISTS [key]`

获取所有键值对

`KEYS *`

获取所有以xx结尾的键值对

`KEYS *xx`

删除所有键值对

`FLUSHALL`

设置键值对过期时间

`EXPIRE [key] [seconds]`

获取键值对过期时间

`TTL [key]`

设置键值对并设置过期时间

`SETEX [key] [seconds] [value]`

设置不存在的键值对(若已存在则忽略执行)

`SETNX [key] [value]`

##### List

创建列表并添加元素

`LPUSH [list] [value, ...]`

`RPUSH [list] [value, ...]`

获取整个列表

`LRANGE [list] 0 -1`

删除列表元素

`LPOP [list] [count]`

`RPOP [list] [count]`

获取列表长度

`LLEN [list]`

仅保留索引为n到m的部分列表

`LTRIME [list] n m`

##### Set

创建集合并添加元素

`SADD [set] [value, ...]`

获取集合中的元素

`SMEMBERS [set]`

判断元素是否在集合中

`SISMEMBER [set] [value]`

删除集合中的元素

`SREM [set] [value]`

##### SortedSet

创建有序集合并添加元素

`ZADD [sortedset] [(score value), ...]`

获取有序集合全部元素

`ZRANGE [sortedset] 0 -1 WITHSCORES`

获取有序集合某个元素权值

`ZSCORE [sortedset] [value]`

获取有序集合某个元素排名

`ZRANK [sortedset] [value]`

`ZREVRANK [sortedset] [value]`

删除有序集合某个元素

`ZREM [sortedset] [value]`

##### Hash

创建哈希集合

`HSET [hash] [key] [value]`

获取哈希集合中的元素

`HGET [hash] [key]`

获取哈希集合中的全部元素

`HGETALL [hash]`

删除哈希集合中的某个元素

`HDEL [hash] [key]`

判断哈希集合中的某个元素是否存在

`HEXISTS [hash] [key]`

获取哈希集合中的键

`HKEYS [hash]`

获取哈希集合长度

`HLEN [hash]`

#### 发布订阅模式

订阅频道

`SUBSCRIBE [channel]`

发布频道消息

`PUBLISH [channel] [message]`

##### Stream

向消息队列中添加一条消息

`XADD [stream] * [key] [value]`

获取消息队列中消息数量

`XLEN [stream]`

获取消息队列中的所有消息

`XRANGE [stream] - +`

删除消息队列中的某条消息

`XDEL [stream] [id]`

删除消息队列中的所有消息

`XTRIM [stream] MAXLEN 0`

读取消息队列中索引为i开始的n条消息，如不存在阻塞x秒

`XRED COUNT n BLOCK x STREAMS [stream] i`

创建消费者组

`XGROUP CREATE [stream] [group] [id]`

获取消费者组信息

`XINFO GROUPS [stream]`

向消费者组中添加消费者

`XGROUP CREATECONSUMER [stream] [group] [consumer]`

读取消费者组中某消费者的最新n条消息，如不存在阻塞x秒

`XREDGROUP [group] [consumer] COUNT n BLOCK x STREAMS [stream] >`

#### 事务

注: Redis中的事务概念与传统事务概念不同，不保证原子性


标记一个事务开始

`MULTI`

提交事务

`EXEC`

#### 持久化

Redis有RDB(Redis Database)和AOF(Append Only File)两种数据持久化方式。

##### RDB

RDB相当于数据快照，可在redis.confg文件中配置自动触发，例如<u>save 60 10000</u>代表60s内如果有10000次修改则触发一次快照，也可通过CLI中的save命令手动触发。

##### AOF

AOF相当于使用日志记录操作命令，可在redis.confg文件中配置参数<u>appendonly yes</u>自动触发。

#### 主从复制

Redis主从复制是指将Redis主节点数据复制到从节点，数据的复制是单向的。

假设主节点端口号为6379，从节点端口号为6380。在从节点的redis.confg文件中配置参数<u>replicaof 127.0.0.1 6379</u>启用主从复制功能。启动从节点6380服务后，使用CLI命令

`redis-cli -p 6380`

`info replication`

即可看到当前节点角色已从master变为slave。

#### 哨兵模式

Redis主从复制存在一个问题，如果主节点宕机了，需要手动去将从节点设置为主节点。为了实现主节点的自动故障转移，Redis引入了一个独立的进程来监视主节点，通过发布订阅模式通知从节点变更为主节点。

创建配置文件sentinel.conf并配置参数<u>sentinel monitor master 127.0.0.1 6379 1</u>(1代表故障转移需要同意的哨兵的个数)，使用CLI命令

`redis-sentinel sentinel.conf`

即可启动哨兵进程。

## 参考文档

[Redis CLI命令官方文档](https://redis.io/docs/latest/commands/)