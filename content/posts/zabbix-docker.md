---
title: Docker部署zabbix填坑指南
categories: ["Docker"]
tags: ["docker","docker-compose","zabbix"]
date: 2020-02-11 20:54:41
---

Zabbix 是由 Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。可用于监视各种网络服务、服务器和网络机器等状态。这里我简单写一下自己在使用 docker 部署 zabbix 中遇到的坑。

<!--more-->

官网的例子比较复杂，包含多种代理，以及一些不需要用到的景象，我在官网的 [docker-compose.yml](https://github.com/zabbix/zabbix-docker/blob/4.4/docker-compose_v3_alpine_mysql_latest.yaml) 文件基础上，做出了一些修改。官网的例子是把 zabbix agent 和 server 端放在一起的。但是在平时应用中，这种情况应该比较少，所以本文的示例，是将他们分开部署的。

### 网桥创建

由于本机测试需要在多个容器间通信，所以这里我建立一个虚拟网桥来共享网络。使用如下命令：

```
docker network create zabbix
```

创建了名为 zabbix 的网桥：

```
docker network ls
```

即可查看所有的网桥.

#### tips

* 查看使用本网桥的容器：

```
docker network inspect zabbix
```

### docker-compose文件编写

这里的服务包含三个：

* zabbix-db：用于存储 zabbix 必须的后台数据。
* zabbix-server-mysql：zabbix 服务端，用于管理各种监控，是 zabbix 的核心。
* zabbix-web-nginx-mysql：提供一个可视化的 web 界面来调试以及配置各项监控指标。

```
# cat docker-compose.yml
version: "3"
services:
  zabbix-db:
    image: mysql:5.7
    container_name: zabbix-db
    environment:
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix
      - MYSQL_ROOT_PASSWORD=12345678
    volumes:
      - "${PWD}/mysql:/var/lib/mysql"

  zabbix-server-mysql:
    image: zabbix/zabbix-server-mysql:latest
    container_name: zabbix-server-mysql
    environment:
      - DB_SERVER_HOST=mysql
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix
      - MYSQL_ROOT_PASSWORD=12345678
    ports:
      - "10051:10051"
    links:
      - zabbix-db:mysql
    depends_on:
      - zabbix-db

  zabbix-web-nginx-mysql:
    image: zabbix/zabbix-web-nginx-mysql:latest
    container_name: zabbix-web-nginx-mysql
    environment:
      - DB_SERVER_HOST=mysql
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix
      - MYSQL_ROOT_PASSWORD=12345678
      - TZ=Asia/Shanghai

    ports:
      - "8081:80"
    links:
      - zabbix-db:mysql
      - zabbix-server-mysql:zabbix-server
    depends_on:
      - zabbix-server-mysql

networks: 
  default:
    external: 
      name: zabbix
```



这一步，有三点需要注意：

1. docker-compose 的 links ：

   ```
   links:
   	- zabbix-db:mysql
   ```

   如果给 links 中的service设置了别名，那么在环境变量中使用的时候，也应该使用别名，不然无法识别。

2. 网桥设置

   由于我们这里需要需要单独给 zabbix 设置网络，所以配置了 networks。

   ```
   networks: 
     default:
       external: 
         name: zabbix
   ```

   上述配置的意思是，当前 docker-compose 所属的 service 都默认使用外部网桥 zabbix。

3. Zabbix 的 mysql 容器，必须指定为数据库为 `zabbix`，因为 zabbix 初始化的时候，有很多配置都是在这个里面完成的。

然后`docker-compose up -d` 启动即可。

   

   

### zabbix-agent 配置

   agent 是属于 zabbix 的一个监控单元，一个 zabbix server 可以对应多个监控 agent ，每个 agent 都可以在 web 界面上，配置多个监控模板。这里主要说明一下如何用 docker 启动。

首先需要查询 zabbix-server 的容器ip：

```
docker exec -it zabbix-server-mysql ip addr
```

output：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN qlen 1000
    link/tunnel6 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
453: eth0@if454: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:15:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.21.0.3/16 brd 172.21.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

`172.21.0.3` 即是 zabbix-server 的容器内部 ip 地址。

 启动 zabbix-agent：

```
docker run --name zabbix-agent-a -p 10050:10050 --network zabbix  -e ZBX_SERVER_HOST=172.21.0.3 -e ZBX_SERVER_PORT=10051 -d zabbix/zabbix-agent
```

* --network zabbix 的意思是将这个容器加入之前的 docker-compose 的同个网段中。
* ZBX_SERVER_HOST,对应 zabbix-server 的服务器ip，由于我是本机测试，所以这里写刚才的`172.21.0.3`。
* ZBX_SERVER_PORT=10051 是 server 的端口。

### 结语

>如果需要在多台机器上创建 zabbix-agent，使用上述命令即可，不过 zabbix server ip 需要修改为服务端所在的服务器的 ip，而且也不再需要网桥的设置了。

zabbix的按照方式有很多种，我这里的`docker-compose` 文件并未展现zabbix的所有功能，详细的功能插件以及如何使用zabbix去监控各项指标，还请查看官方文档。

