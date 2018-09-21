---
title: tomcat启动报错
categories: Java
tags: [java,tomcat]
date: 2018-9-21 23:01:41 
author: gravel
---

tomcat 启动报错
```
ClassCastException: org.apache.tomcat.websocket.server.WsServerContainer cannot be cast to javax.websocket.server.ServerContainer
```

### 问题原因
出现这个问题的原因是，apache的websocket包和javax-servlet包冲突了。

### 解决方案
在pom文件中，排除javax.websocket
```
<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
    <version>1.0</version>
    <scope>provided</scope>
</dependency>
```