---
title: docker安装目录迁移
categories: ["Docker"]
tags: ["docker"]
date: 2018-10-20 23:01:41 
author: gravel
---

`docker`默认安装路径是`var/lib/docker`这个目录下面的，如果这个目录挂载的空间不大的话，那么在实际使用中，可能会导致`docker`空间不足的情况，我们需要将这个默认目录迁移到比较大的空间下面去。

<!--more-->

### 基础环境
* centos 7
* docker 1.18

### 执行步骤

* 停止`docker`

```
systemctl stop docker
```

* 创建新的`docker`安装目录，我的机器上，`home`目录空间比较大，所以我选择了这个目录
```
mkdir -p /home/lib/docker
```
* 将现有安装目录，辅助到刚刚创建的目录
```
cp -R /var/lib/docker/* /home/lib/docker/
```

* 修改`docker`配置（`/etc/systemd/system/docker.service.d/devicemapper.conf`），如果没有此目录或者文件，需要自己重新创建。我的机器没有，所以：
```
mkdir -p /etc/systemd/system/docker.service.d/;

vi devicemapper.conf
```

在文件末尾，写入以下内容：
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --insecure-registry=私服地址 --graph=/home/lib/docker
```

 注意：如果没有私服地址的话就可以去掉`”--insecure-registry=私服地址”`。

 * 然后重启`docker`
```
systemctl daemon-reload //重载进程

systemctl restart docker // 重启docker
```

* 然后`docker info` 检查以下是否已经修改完成。
```
Docker Root Dir: /home/lib/docker/
```