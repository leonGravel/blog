---
title: azkaban回调配置
categories: ["azkaban"]
tags: ["java","调度"]
date: 2019-07-06 23:01:41 
author: gravel

---

`azkaban` 是`linkin`开源的一套简单的任务调度服务系统。如果需要配置任务的状态回调，那么需要加入以下配置：

```
type=command

job.notification.started.1.url = http://10.20.115.20:9527/index/callback?message=started&server=?{server}&project=?{project}&flow=?{flow}&executionId=?{executionId}&job=?{job}&status=?{status}

job.notification.success.1.url = http://10.20.115.20:9527/index/callback?message=success&server=?{server}&project=?{project}&flow=?{flow}&executionId=?{executionId}&job=?{job}&status=?{status}

job.notification.failure.1.url = http://10.20.115.20:9527/index/callback?message=failure&server=?{server}&project=?{project}&flow=?{flow}&executionId=?{executionId}&job=?{job}&status=?{status}

job.notification.completed.1.url = http://10.20.115.20:9527/index/callback?message=completed&server=?{server}&project=?{project}&flow=?{flow}&executionId=?{executionId}&job=?{job}&status=?{status}

command=exit -1

dependencies=callback


```

其中主要配置是`job.notification.xxxxx.1.url`。   