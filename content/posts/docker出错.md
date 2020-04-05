---
title: Docker上传镜像到私服出错
categories: ["Devops"]
tags: ["docker","问题笔记"]
date: 2018-09-05 23:01:41 
author: gravel
---

今天在公司将自己的镜像上传到公司仓库的时候，出现了错误：

<!--more-->

```
Get https://xxx.xxxxxx.xxxx:5000/v2/ http: server gave HTTP response to HTTPS client
```
### 问题原因
出现这问题的原因是：Docker自从1.3.X之后docker registry交互默认使用的是HTTPS，但是搭建私有镜像默认使用的是HTTP服务，所以与私有镜像交时出现以上错误。
### 解决方法
因为公司电脑是用的windows，所以打开settings>Daemon，将Advanced现象打开，在下方的json格式的文件里，将你的私服地址，填写在`insecure-registries`对应的地方。
```
 "insecure-registries": [
    "xxx.xxxxxx.xxxx:5000"
  ],
```
### 结束
不得不说，windows下面玩Docker，坑还是有点多的