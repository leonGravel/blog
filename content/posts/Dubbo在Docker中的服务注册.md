---
title: Dubbo在Docker中的服务注册
categories: Java
tags: [docker,dubbo]
date: 2018-09-28 23:01:41 
author: gravel
---
今天在部署的时候，发现服务提供者启动成功，但是消费者没法拿到服务。查了下发现是服务提供者没注册成功。

<!--more-->

### 问题分析
当docker容器部署dubbo提供者和常规部署应用混合使用一套zookeeper时，将出现Docker容器中的dubbo提供者向zookeeper注册容器IP导致常规部署应用无法访问容器IP而失败。
### 解决方案
在github上查的时候，发现有人提出了这个[问题][1]，官方回复是将在2.5.7的版本中解决。
Dubbo在启动阶段提供两对系统属性，用于设置外部通信的IP和端口地址。
* DUBBO_IP_TO_REGISTRY --- 注册到注册中心的IP地址
* DUBBO_PORT_TO_REGISTRY --- 注册到注册中心的端口
* DUBBO_IP_TO_BIND --- 监听IP地址
* DUBBO_PORT_TO_BIND --- 监听端口

我将启动命令改为如下格式：
```
docker run -d \
    --name <containerName> \
    --net dubbo \
    -e DUBBO_IP_TO_REGISTRY=<ip> \
    -e DUBBO_PORT_TO_REGISTRY=<port> \
    -p <ip>:<port>:<port> \
    -v dubbo:/log \
    --restart=always \
    <imageName> 
```

问题得到解决！

[1]: https://github.com/apache/incubator-dubbo/issues/668