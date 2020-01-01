---
title: 给Docker容器设置固定ip
categories: ["Docker"]
tags: ["docker","docker-compose"]
date: 2019-02-11 20:54:41


---

今天在查应用日志的时候，发现日志收集分析的应用，收到很多`ip`发来的同一系统的日志。经分析发现，这么多ip都是出自三台机器，由于过年期间有些机器有过断电重启的情况，所以 `docker` 给这个应用重新赋予了`ip`，导致了以上的情况出现，日志分析应用，无法知道这个日志的准确来源。

<!--more-->

在处理这个问题之前，首先要知道的是，`docker`有三种网络类型。

* **bridge：桥接网络**

默认情况下启动的Docker容器，都是使用 `bridge`，`Docker`安装时创建的桥接网络，每次`Docker`容器重启时，会按照顺序获取对应的`IP`地址，这个就导致重启下，`Docker`的`IP`地址就变了

* **none：无指定网络**

使用 `--network=none `，`docker` 容器就不会分配局域网的`IP`

* **host： 主机网络**

使用 `--network=host`，此时，`Docker` 容器的网络会附属在主机上，两者是互通的。

例如，在容器中运行一个`Web`服务，监听8080端口，则主机的8080端口就会自动映射到容器中。



由上述可知，我的应用是使用第一种的默认情况。在应用重启，或者`docker`重启之后，`ip`会发生变化。此应用是由`docker-compose`部署启动的，所以这里我只说明一下`docker-compose`的解决方案。

在`docker-compose.yml`文件里面，按照如下格式调整：

```
version: '3'

services:
  app:
    image: myapp
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
```

