---
title: 引入spring-cloud-starter-openfeign后有些类找不到 
categories: ["java"]
tags: ["java","springcloud"]
date: 2018-09-03 23:01:41 
author: gravel
---
今天用ali maven重新导入的spring-cloud-starter-openfeign

<!--more-->

![enter description here][1]
我用的aliyuMaven仓库，发现spring-cloud-starter-openfeign与spring官方仓库的pom配置不一样。
![enter description here][2]

下面是ali
![enter description here][3]

重新切回官方镜像仓库之后，这个问题得到解决。


[1]: 1.png "1"
[2]: 2.png "2"
[3]: 3.png "3"