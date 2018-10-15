---
title: docker-compose搭建flink环境
categories: Docker
tags: [docker,docker-compose]
date: 2018-10-15 23:01:41 
author: gravel
---

嗯，最近在搭建flink的环境，用普通的docker命令构建的时候，老是遇到各种的问题。于是转为用docker-compose试试。
### docker-compose 介绍
docker-compose 是一个用来把 docker 自动化的东西。
有了 docker-compose 你可以把所有繁复的 docker 操作全都一条命令，自动化的完成。
用通俗的语言来说，我们平时操作 docker 还是很原始的一系列动作，你手动使用 docker 的动作可以拆分成：

> 1. 找到一个系统镜像 // docker search
> 2. 安装好 vm 或者 virtual box // apt-get install docker
> 3. 在 vm 中安装镜像 // docker run -d -it 你的镜像

这是最小的动作， 如果你要映射硬盘，设置nat网络或者桥接网络，等等…你就要做更多的 docker 操作， 这显然是非常没有效率的。

但是我们写在 `docker-compose.yaml` 里面就很好了。 你只需要写好后 只运行一句
`docker-compose up -d` 就可以启动了。

### 安装docker-compse

下载最新版的docker-compose文件 
```
udo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
添加可执行权限 
```
sudo chmod +x /usr/local/bin/docker-compose
```
测试安装结果 

```
docker-compose --version 
```

### 安装flink

1. 在指定目录下，新建docker-compose.yml 文件如下：
```
version: "2.1"
services:
  jobmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6123"
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6121"
      - "6122"
    depends_on:
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
```

文件的意思是，先基于flink镜像，启动一个jobmanager，然后再基于jobmanager和flink镜像，启动一个taskmanager.

新建完成之后，在当前目录`docker-compose up`.然后访问localhost:8081查看结果，如果taskmanager页面有数据。说明flink已经部署成功。