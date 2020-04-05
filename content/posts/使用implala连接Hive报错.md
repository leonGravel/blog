---
title: 使用implala连接hive报错
categories: ["Python"]
tags: ["Python","问题笔记"]
date: 2019-07-01 20:54:41
---

今天在使用`implala` 连接 `hive` 数据库的时候，出现了一个错误。

<!--more-->

`implala` 各种依赖安装好之后，测试连接时，报了以下错误：

```
TypeError: can't concat str to bytes
```

定位到问题，位于`thrift_sasl`这个包的`init.py`第93行：

```
header = struct.pack(">BI", status, len(body))
self._trans.write(header + body)
```

这里的`body`可能是`str`类型，所以抛出了这个异常。然后我去github上搜索了一下源码。发现作者竟然已经放弃维护了。于是只好自己动手，很简单，在94行加入以下代码：

```
header = struct.pack(">BI", status, len(body))
if(type(body) is str):
    body = body.encode() 
self._trans.write(header + body)
```

有需求的话，可以自己下载源码编译。点[这里](https://github.com/leonGravel/thrift_sasl/pull/1)。