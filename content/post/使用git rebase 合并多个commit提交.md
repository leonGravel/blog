---
title: 使用git rebase合并多个commit提交
categories: ["git"]
tags: ["git"]
date: 2019-02-27 23:01:41 
author: gravel

---

在日常开发中，常常会遇到自己正在开发某个`feature`的时候，需要切到另外的分支去处理`bug`。于是先将未完成的功能`commit`到本地。处理完`bug`之后，再切回来开发，这种做法有一个坏处是，仓库`commit`的历史会很凌乱。不利于追踪排查历史问题。

<!--more-->

以上为前提条件，这种情况可以使用 `git rebase` 来处理，合并多个本地的`commit`，今天以这边文章的提交历史，来做个示范。

>注意，这种合并方式，只对未`push`到远端的`commit`有效

### 查看git log

首先查看下这个仓库的提交历史记录：

```
git log 
```

输出如下图：



### 