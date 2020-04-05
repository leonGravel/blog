---
title: 拷贝文件到Docker容器中
categories: ["Devops"]
tags: ["docker"]
date: 2018-09-27 23:01:41 
author: gravel
---
查找需要拷贝的目的地容器

<!--more-->

```
docker ps -a
```
找出我们想要的容器名字全称 
查找容器长ID

```
docker inspect -f '{{.ID}}' 容器名字
```

然后通过容器ID拷贝

```
docker cp 本地路径 容器长ID:容器路径

docker cp /var/gravel/config.properties 38ef22f922704b32cf2650407e16b146bf61c221e6b8ef679989486d6ad9e856:/root/web/config.properties

```