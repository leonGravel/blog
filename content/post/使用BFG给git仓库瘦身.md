---
title: 使用BFG给git仓库瘦身
categories: ["git"]
tags: ["git","bfg.jar"]
date: 2018-11-01 23:01:41 
author: gravel

---

今天遇到一个问题，如何给一个gitlab的仓库瘦身。在我们日常开发中，由于不规范或者不小心，误提交了一些大文件，导致git的仓库变得很大，这是你直接删除大文件也无济于事，因为git commit log里面，会记录你每一次的提交详情。一般来说，给git瘦身有两种方式，一种是官方提供的`git-filter-branch`,这种命令用起来极为繁琐。另一种是本文将要说到的BFG.

<!--more-->

### BFG介绍

在gitlab的帮助页面中也推荐了这个工具。官网说是比git-filter-branch工具快10-720倍。这里根据我的使用，介绍一下这个工具。

这个工具的官网：<https://rtyley.github.io/bfg-repo-cleaner/>

### 使用步骤

1. 下载官网的程序包。重命名为bfg.jar
2. clone自己的git repo，使用--mirror参数。(注意这里一定要加`--mirror` 参数，mirror 可以保证本地仓库和远端完全一致)

```
git clone --mirror git@github.com:repo.git
```

3. 将bfg.jar放到和repo.git同级的目录

```
java -jar bfg.jar --strip-blobs-bigger-than 1M repo.git
```

这一步的目的是，删除commit历史中，文件大小大于1M的二进制文件。

4. 使用git gc清理不需要的数据

```
cd repo.git
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

5. 提交更改

```
git push
```

