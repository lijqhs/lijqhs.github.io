---
title: Docker启动Redis, Low Memory Warning处理方法
slug: redis-memory-overcommit
date: 2023-04-08
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-louvre.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-louvre.jpg
categories:
- dev
tags:
- redis
---


Redis是一种高性能的开源内存数据结构存储系统，可以用作数据库、缓存和消息代理。它支持很多数据结构，包括字符串、哈希、列表、集合、有序集合等。Redis在内存中存储数据，并且支持数据持久化，可以将数据存储在磁盘上，以便在重启时恢复数据。

<!--more-->

{{< toc >}}


## Redis常见使用场景

Redis的高性能和可扩展性使其在许多场景下都有广泛的应用，例如：

- 缓存：Redis可以作为缓存层，缓存热门数据，从而减少数据库的访问压力。Redis的高性能和灵活性使其成为用户最受欢迎的缓存之一。
- 分布式锁：Redis支持分布式锁，可以在分布式环境下保证数据的一致性和可靠性。
- 计数器：Redis可以用作计数器，支持原子操作，可以实现高并发下的计数器功能。比如，用于电商网站商品的浏览量或者视频网站的播放量计数等等。
- 排行榜：Redis的有序集合可以用来实现排行榜功能。
- 实时聊天：Redis的发布/订阅功能可以用来实现实时聊天室。

## Redis核心技术优势

- 高性能：Redis将所有数据存储在内存中，因此可以实现非常高的读写性能。Redis还使用了一些优化技术，如异步IO、多路复用等，进一步提高了性能。
- 多种数据结构：Redis支持多种数据结构，包括字符串、哈希、列表、集合、有序集合等，可以满足各种需求。
- 数据持久化：Redis支持数据持久化，可以将数据存储在磁盘上，以便在重启时恢复数据。
- 高可用性：Redis支持主从复制和哨兵模式，可以实现高可用性和容错性。
- 分布式：Redis支持分布式，可以将数据分散到多个节点上，从而实现扩展性和负载均衡。

## Redis竞争产品

Redis有一些竞争产品，例如Memcached、Riak、Cassandra等。这些产品也具有高性能、可扩展性和分布式等特点，但是它们可能具有不同的特点和适用场景。

## 用Docker来部署Redis，常见问题

使用Docker来部署Redis非常简单，可以避免复杂的安装过程。以下是一些常见问题的解决方案：

- 如何启动Redis容器：可以使用以下命令启动Redis容器：`docker run -d --name redis -p 6379:6379 redis`
- 如何连接到Redis容器：可以使用以下命令连接到Redis容器：`docker exec -it redis redis-cli`
- 如何持久化数据：可以使用以下命令启动Redis容器，并将数据存储在本地目录中：`docker run -d --name redis -p 6379:6379 -v /path/to/data:/data redis redis-server --appendonly yes`
- 如何设置密码：可以使用以下命令设置密码：`redis-cli config set requirepass yourpassword`

另外一个常见问题是在启动Docker容器时，常常会收到如下报警：

> WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

根据提示，需要在Docker主机中将`vm.overcommit_memory`设为`1`。用文本编辑器打开`/etc/sysctl.conf`文件，在文件末尾添加以下一行：
```
vm.overcommit_memory=1
```
保存更改并退出文本编辑器。然后重启主机，或者执行命令`sysctl vm.overcommit_memory=1`激活设置。

### overcommit_memory

`overcommit_memory`定义了决定是否接受或拒绝大内存请求的条件。
- 默认值是`0`。默认情况下，内核通过估计可用的内存量和拒绝过大的请求来执行启发式的内存超限处理。然而，由于内存的分配是使用启发式的，而不是精确的算法，所以使用这个设置有可能造成内存超载。
- 当这个参数被设置为`1`时，内核不执行内存超载处理。这增加了内存超载的可能性，但是提高了内存密集型任务的性能。
- 当该参数设置为`2`时，内核拒绝对等于或大于总的可用交换空间和`overcommit_ratio`中指定的物理RAM百分比的内存的请求。这减少了过度提交内存的风险，但只推荐给交换区大于物理内存的系统。


{{< alert info >}}
参考文章：
- https://redis.io/docs/management/admin/
- https://www.kernel.org/doc/Documentation/vm/overcommit-accounting
- https://serverfault.com/questions/606185/how-does-vm-overcommit-memory-work
{{< /alert >}}


## 学习Redis的资源

学习Redis的资源有很多，以下是一些推荐的资源：

- [Redis官方文档](https://redis.io/documentation)
- [Redis源代码](https://github.com/redis/redis)
- [Redis实战（第二版）](https://book.douban.com/subject/30386804/)
- [Redis设计与实现](https://book.douban.com/subject/25900156/)
- [Redis教程](https://www.runoob.com/redis/redis-tutorial.html)

总之，Redis是一个功能强大、高性能、可靠的键值存储系统，被广泛使用于各种应用程序中，成为了当今最流行的NoSQL数据库之一。
