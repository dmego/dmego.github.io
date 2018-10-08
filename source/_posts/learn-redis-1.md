---
layout: post
title: 'Redis系列(一):Redis的简介与安装'
comments: true
date: 2018-10-4 2:02:31
categories: [DataBase]
tags: [Redis,学习笔记]
---

<!--more -->

<center>
<img src="learn-redis-1\redis-white.png" width="250px" />
</center>


## 什么是 Redis

Redis 是一个使用`ANSI C` 编写的开源、支持网络协议、基于内存、可选持久性的键值对数据库,它是一个`NOSQL`not only sql)数据库,也就是常说的非关系型数据库。从 2005 年开始，Redis 的开发由 Redis Labs 赞助，之前一直被 Pivotal 和 VMware 先后赞助。根据月度排行网站 DB-Engines.com 的数据显示，Redis 是最流行的键值对数据库。

## Redis 应用场景

主要用于数据量大，并发量高的情况下

- 数据缓存（页面具体数据），页面缓存（商品内容，新闻内容）
- 分布式集群中架构中的Session分离
- 应用排行榜，在线好友列表等
- 任务队列，例如抢购秒杀等

## Redis 的安装

Redis 是不支持 windows 版本的，因为其在 windows 下的效率非常低，但是Microsoft 开放技术小组开发和维护了一个针对 [windows 版的 Redis](https://github.com/MicrosoftArchive/redis),但是从项目记录来看已经有两年没更新了。

从[官网](https://redis.io/)可以看到，目前最新稳定版为 Redis 4.0.11。此次安装环境选择 centOS ,方式为源码编译安装，所以我们需要有`gcc`环境，先执行下面的命令安装 `gcc`

```bash
yum install gcc-c++ -y
```

然后将最新稳定版的源码包下载下来

```bash
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
```

解压到`/usr/local/`目录下

```bash
tar -zxvf redis-4.0.11.tar.gz -C /usr/local
```

进入Redis 目录进行编译

```bash
cd /usr/local/redis-4.0.11
make
```

编译完成后，安装到指定目录，例如:/usr/local/redis-6379（也可以直接命令`6379`，因为可以在同一台机器上运行多个 redis 服务，所以一般以运行端口命名）

```bash
cd /usr/local/redis-4.0.11
make PREFIX=/usr/local/redis-6379 install
```

安装完成之后，我们还需要拷贝一份 redis 的配置文件——`redis.conf` 到安装路径下面，`redis.conf` 在 redis 的源码目录下

```bash
cd /usr/local/redis-4.0.11
cp redis.conf /usr/local/redis-6379/bin/
```

![安装目录bin下的文件列表](learn-redis-1\redis安装后目录结构.png)

| 文件名         | 说明    |
| :------------ | ------- |
| redis-server | redis 服务器 |
| redis-cli | redis 命令行客户端 |
| redis-benchmark | redis 性能测试工具 |
| redis-check-aof | aof 文件修复工具 |
| redis-check-dump | rub 文件检查工具 |

## Redis 的启动

#### 前端模式启动

如果在`bin`目录下直接运行 `./redis-server`将以前端模式启动,启动成功界面如下所示

![redis前端启动](learn-redis-1\redis前端启动.png)

这种方式启动后，我们不能关闭该窗口，关闭该窗口后 redis 服务将会停止。如果想要使用 redis 需要再开一个窗口。进入到`bin`目录，运行`./redis-cli`命令，开启一个 redis 客户端连上 redis 服务。

#### 后端模式启动

我们进入`bin`目录，先给之前拷贝过来的`redis.conf`配置文件赋予权限

```bash
cd /usr/local/redis-6379/bin/
chmod 777 redis.conf
```

然后打开 `redis.conf`配置文件，修改启动参数`daemonize`为`yes`，以后端方式启动。如果找不到`daemonize`,可以使用 `vim`打开文件后，使用`:/daemonize`来查找，找到后修改`no`为`yes`，然后`wq`保存退出。此时，我们就可以使用如下命令以后端方式启动 redis

```bash
./redis-server redis.conf
```

启动后可以使用`ps -aux|grep redis`命令来查看是否启动成功

![后端启动成功](learn-redis-1\后端启动成功.png)

如上图所示,默认启动端口为 `6379`

## Redis 的停止

如果我们强行停止 Redis 的进程可能会导致 Redis 持久化的数据丢失，所以正确停止 Redis 的方式应该是使用 `./redis-cli shutdown`命令。

## Redis 客户端的连接和使用

在`bin`目录下的`redis-cli`就是 redis 的客户端，执行`./redis-cli`命令将会连接到 redis 服务器。连接成功后，我们可以使用`set key1 111`来保存一个值为`111`名为`key1`的键值对。使用`get key1`命令读取`key1`的数据

![连接redis客户端](learn-redis-1\连接redis客户端.png)



## 使用Redis 可视化工具

如果想要在Windows、Mac 或Linux 图像界面下可视化操作 Redis ，可以使用 [Redis Desktop Manager](https://github.com/uglide/RedisDesktopManager/)，但是现在对于Windows 和 Mac 不提供下了，不过在网上还能搜到下载的链接。

![连接可视化工具](learn-redis-1\连接可视化工具.png)

另外还有几款开源的可视化工具也不错，例如：[Redis Client](https://github.com/caoxinyu/RedisClient)、[Redis Studio](https://github.com/cinience/RedisStudio)，但是都已经停止更新维护了。

