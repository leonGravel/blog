---
title: minio设置永久下载链接
categories: ["Devops"]
tags: ["docker","minio"]
date: 2018-12-05 10:01:41 
author: gravel
---

今天在构建[ `oraclejdk7` 镜像](https://hub.docker.com/r/leebroncc/alpine-oraclejdk7)的时候，为了方便自己以后能够随时编译 `jdk` 镜像，于是将 `oracle-jdk-7` 的 `tar.gz` 包上传到了 `minio` 服务器上，可以直接使用 `wget` 命令随时下载。

<!--more-->

在实际操作的时候，发现 `minio` 分享文件，最多支持分享七天，这显然和我的需求有冲突。查看了 `minio` 的文档，发现 `minio` 的功能远比我想象的强大，他提供了一个客户端工具。可以直接对 `minio server` 进行配置。下面我具体说下 `minio` 客户端是怎么设置永久下载链接的。

### 安装客户端

首先，当然是安装客户端，我最开始的时候，使用的 `Docker` 安装，但我发现 `docker` 还需要配置数据卷这些，命令很长，用起来有点麻烦。正好服务器上有 `Go` 的环境，于是直接用 `Go` 命令获取 `minio` 客户端二进制安装文件进行下载了。

```
go get -d github.com/minio/mc
```

然后编译

```
cd ${GOPATH}/src/github.com/minio/mc
make
```

设置权限

```
chmod +x mc
```

设置自定义命令

```
alias mc="${GOPATH}/src/github.com/minio/mc/./mc"
```

至此，我们的 `minio client` 就安装完成了。

### 添加minio host

使用 `minio client` 将我自己的 `minio server` 添加到 `mc` 的配置管理：

```
mc config host add minio http://142.4.xxx.198:9000 minio password S3v4
```

这样我们才能直接管理这个 `minio server` 端。

### 配置下载策略

```
mc policy public minio/base
```

这个命令的作用是将 `server` 端的 `base` 文件设置为开放管理，可以直接通过 `url` 进行下载。

类似于以下 `http://142.4.211.198:9000/base/jdk-7u80-linux-x64.tar.gz` 。

大致的操作流程就是这样，具体可查看[官网文档](https://docs.minio.io/cn/minio-client-complete-guide.html#config) 。
