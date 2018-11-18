---
title: 使用wget下载minio中的内容报错
categories: ["Docker"]
tags: ["docker","minio"]
date: 2018-10-30 23:01:41 
author: gravel
---

### 问题描述
在构建jdk镜像的时候，由于无法直接从oracle上获取到jdk的压缩包，所以我把压缩包放到minio上，通过wget下载（实际上是多此一举，可以直接ADD进去）。但是我在用wget下载minio的数据的时候，直接报错了。

<!--more-->

```
Access to files denied 
```
### 解决办法
在启动minio容器的时候，指定PUBLIC_URL，
具体命令如下：
```
docker run -d  -p 9000:9000 --name minio \
  -e "MINIO_ACCESS_KEY=minio" \
  -e "MINIO_SECRET_KEY=123456@minio" \
  -e "PUBLIC_URL=95.555.9.60" \
  -v /mnt/data:/data \
  -v /mnt/config:/root/.minio \
  minio/minio server /data
```