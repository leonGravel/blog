---
title: Docker容器扩容
categories: Docker
tags: [docker,问题收集]
date: 2018-10-9 23:01:41 
author: gravel
---
Docker默认空间大小分为两个，一个是池空间大小，另一个是容器空间大小。

池空间大小默认为：100G

容器空间大小默认为是：10G

所以修改空间大小也分为两个。

首先关闭docker:

```
systemctl stop docker
```

然后删除docker数据：

```
rm -rf /var/lib/docker
```
创建新的docker数据池:

```
mkdir -p /var/lib/docker/devicemapper/devicemapper

dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=1000
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/metadata bs=1G count=0 seek=10

```
上面的1000为1TB大小，即为数据池空间大小为1TB，而10则为Metadata的空间大小，10GB

重启容器即可完成扩容：

```
systemctl restart docker
```
