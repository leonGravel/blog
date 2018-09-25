---
title: Docker 如何进入运行的tomcat容器
categories: Docker
tags: [docker,tomcat]
date: 2018-9-26 23:01:41 
author: gravel
---

### 问题描述
当docker在 “-d”守护态运行tomcat容器的时候，，docker attach 容器id 就会一直卡着。
因为此时容器运行的进程是ssh，而不是/bin/bash 也没有虚拟终端（-it）参数，所以是进入不到的。

### 解决方案
放弃attach，使用docker exec进入Docker容器
```
sudo docker exec -it {容器id} /bin/bash  
```
使用这条命令即可进入运行的容器。