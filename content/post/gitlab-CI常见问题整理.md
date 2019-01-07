---
title: gitlab-CI常见问题整理 
categories: ["CI"]
tags: ["gitlab","CI"]
date: 2019-01-07 00:01:41
author: gravel
---

整理一下自己在工作中踩过的关于`gitlab-CI`的坑。

<!--more-->

### gitlab提交代码触发构建报错

主要日志如下：

```
unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@gitlab.xxx.com/jd/XXX.git/': Could not resolve host: gitlab.xxx.com
```

我的第一反应是`token`可能失效了，然后我重新注册了一个`runner`与此项目进行关联。 重试之后发现问题仍然存在。联想到最近虚拟机曾经异常关闭过，重启之后，可能防火墙又重新打开了。

于是：

```
systemctl status firewalld
```

查看防火墙状态，果然处于开启状态。猜想应该是防火墙的原因导致`runner`所产生的容器中，无法直接获取到`gitlab`的项目。

```
systemctl stop firewalld
```

关闭防火墙，重启`docker`，然后再次触发，问题得到解决。