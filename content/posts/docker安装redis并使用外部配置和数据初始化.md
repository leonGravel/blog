---
title: docker安装redis并使用外部配置和数据初始化
categories: Docker
tags: [docker,redis]
date: 2018-09-19 23:01:41 
author: gravel
---
我这里安装的是redis:4.0

<!--more-->

### 拉取镜像
首先从镜像仓库拉取Redis的镜像
```
docker pull redis:4.0
```
### 创建挂载目录以及配置文件
在root用户下：
```
mkdir -p /redis/data
```
然后从[redis官网][1]下载和版本对应的redis.conf文件，根据自己的需求修改其中的内容，需要注意的是，一定要把`daemonize yes`给注释掉。
不然启动可能会失败。原因是：

>Redis 进程被幽灵化(后台化)后,启动Redis的那个进程,也就是Docker执行进程无事可做, 因此Docker执行进程退出。

### 启动
使用如下命令启动
```
 docker run -p 6379:6379 -v /root/redis/redis.conf:/etc/redis/redis.conf -v /root/redis/data:/data -d redis:4.0 redis-server /etc/redis/redis.conf

```
这里直接指定我们刚才创建的宿主机挂载目录。
如果需要使用rdb或aof恢复数据或者初始化数据，请一定要在配置文件中指定你说需要的方式，不然会失败。

### 常用命令
这里说一下在docker中怎么使用redis的常用命令
首先，我们需要进入redis容器：
```
 docker exec -it 650 redis-cli
```
其中，650为你的redis容器的id。
输入认证：
```
auth 123456
```
这一步的目的是用密码进入redis。如果顺利的话，你就已经可以在docker容器中使用redis相关的命令了。



[1]: https://github.com/antirez/redis