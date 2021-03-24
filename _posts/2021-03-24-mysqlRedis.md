---
layout: post
title: Mysql与Redis缓存同步方案
date: 2020-03-24
Author: hhw
comments: true
toc: true
tags:  [MySQL,Redis]
pinned: true
---

本文记录MySQL与Redis缓存同步的两种方案，这里只探讨思路

- 普通同步方案
- 通过MySQL自动同步刷新Redis，MySQL触发器+UDF函数实现
- 解析MySQL的binlog实现，将数据库中的数据同步到Redis

### 方案一 普通同步

#### 思路分析

- 读：读redis，没有数据就读mysql，将MySQL数据保存到缓存中。
- 写：写mysql，同时让redis缓存失效（删除key，过期）
- 缺点：如果数据量巨大，更新频繁的数据写入无能为力。比如数量巨大，每个变跟状态又很频繁，这样很容易把数据库写挂。

### 方案二 UDF

#### 思路分析

当我们对MySQL数据库进行数据操作时，同时将相应的数据同步到Redis中，同步到Redis之后，查询的操作就从Redis中查找

#### 过程分析

- 在MySQL中对要操作的数据设置触发器Trigger，监听操作
- 客户端（NodeServer）向MySQL中写入数据时，触发器会被触发，触发之后调用MySQL的UDF函数
- UDF函数可以把数据写入到Redis中，从而达到同步的效果

![图片](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210324174603.png)

#### 适用场景

- 这种方案适合于读多写少，并且不存并发写的场景
- 因为MySQL触发器本身就会造成效率的降低，如果一个表经常被操作，这种方案显示是不合适的



### 方案三 binlog

在介绍方案2之前我们先来介绍一下MySQL复制的原理，如下图所示：![图片](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210324175303.png)

- 主服务器操作数据，并将数据写入Bin log
- 从服务器调用I/O线程读取主服务器的Bin log，并且写入到自己的Relay log中，再调用SQL线程从Relay log中解析数据，从而同步到自己的数据库中

方案2就是：

- 上面MySQL的整个复制流程可以总结为一句话，那就是：从服务器读取主服务器Bin log中的数据，从而同步到自己的数据库中
- 我们方案2也是如此，就是在概念上把主服务器改为MySQL，把从服务器改为Redis而已（如下图所示），当MySQL中有数据写入时，我们就解析MySQL的Bin log，然后将解析出来的数据写入到Redis中，从而达到同步的效果

![图片](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210324175307.png)

例如下面是一个云数据库实例分析：

- 云数据库与本地数据库是主从关系。云数据库作为主数据库主要提供写，本地数据库作为从数据库从主数据库中读取数据
- 本地数据库读取到数据之后，解析Bin log，然后将数据写入写入同步到Redis中，然后客户端从Redis读数据

![图片](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20210324175311.png)

**这个技术方案的难点就在于：** 如何解析MySQL的Bin Log。但是这需要对binlog文件以及MySQL有非常深入的理解，同时由于binlog存在Statement/Row/Mixedlevel多种形式，分析binlog实现同步的工作量是非常大的

阿里巴巴：Canal开源技术：基于数据库增量日志解析，提供增量数据订阅&消费，目前主要支持了MySQL

### 方案四 先存缓存再存数据库

- 客户端有数据来了之后，先将其保存到Redis中，然后再同步到MySQL中
- 这种方案本身也是不安全/不可靠的，因此如果Redis存在短暂的宕机或失效，那么会丢失数据

