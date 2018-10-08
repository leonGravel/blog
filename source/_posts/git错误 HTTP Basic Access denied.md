---
title: git错误 HTTP Basic Access denied
categories: Git
tags: [git,问题收集]
date: 2018-10-8 23:01:41 
author: gravel
---
### 问题症状
修改了git密码之后，拉取项目代码出错：
```
git remote: HTTP Basic: Access denied 
```
### 原因
远程服务端的用户名和密码与当前系统中git保存的用户名和密码有冲突
### 解决方案

```
git config --system --unset credential.helper.
```