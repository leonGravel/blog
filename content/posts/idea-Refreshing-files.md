---
title: idea Refreshing files 卡顿
categories: ["Devops"]#文章文类
tags: ["idea","问题笔记"]
author: gravel
date: 2019-11-18 18:01:41 

---

Idea 切换maven仓库之后，重新构建老是卡住。。如图所示。

![img](https://intellij-support.jetbrains.com/hc/user_images/VBTyrBAX3RZ1HfxV4dx95Q.png)



按照 https://intellij-support.jetbrains.com/hc/en-us/community/posts/360000027164-Refreshing-files-takes-way-tool-long-often-before-building-

的操作提示，File" -> "Invalidate Caches and Restart" 重启之后，得到解决。