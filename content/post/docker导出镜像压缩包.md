---
title: docker导出镜像压缩包
categories: ["Docker"]
tags: ["docker"]
date: 2019-02-18 23:01:41 
author: gravel

---

网上关于`docker`镜像的导出导入的文章已经很多了，无非是`save`、`export` 、`load`、`import` 这几个命令，我这里只是简单记录一下今天遇到的一个特殊情况。

<!--more-->

使用`docker save`命令导出镜像文件的时候，看到的大小是没有压缩过的。我在`google`查到，如果给命令加上`gzip`，那么就会用`gzip`的格式打包镜像。命令如下：

```
docker save myimages:6 | gzip -c > myimages-6.tar.gz
```

这么压缩导出的镜像文件会小很多。不过奇怪的是，`docker`官网上并没有介绍这种方式。