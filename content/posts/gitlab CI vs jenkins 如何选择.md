---
title: gitlab CI vs jenkins 如何选择
categories: ["CI"]
tags: ["Jenkins","gitlab-CI"]
date: 2018-11-04 23:01:41 
author: gravel
---

之前在项目中做过CI的技术选型，基本成熟之后，现在来总结一下两者的优劣。		

<!--more-->

## Jenkins

`Jenkins` 是一个广泛用于持续构建的可视化 `web` 工具，`jenkins` 可以很好的支持各种语言的项目构建，也完全兼容`ant`、`maven`、`gradle`等多种第三方构建工具，同时跟`svn`、`git`能无缝集成，也支持直接与知名源代码托管网站，比如`github`、`bitbucket`直接集成，而且插件众多，在这么多年的技术积累之后，在国内大部分公司都有使用`Jenkins`。

## gitlab-CI

`gitlab-CI`是`gitlab8.0`之后自带的一个持续集成系统，中心思想是当每一次`push`到`gitlab`的时候，都会触发一次脚本执行，然后脚本的内容包括了测试，编译，部署等一系列自定义的内容。

`gitlab-CI`的脚本执行，需要自定义安装对应`gitlab-runner`来执行，代码`push`之后，`webhook`检测到代码变化，就会触发`gitlab-CI`，分配到各个`Runner`来运行相应的脚本`script`。这些脚本有的是测试项目用的，有的是部署用的。

## jenkins VS gitlab-runner

### gitlab-CI优势

* 轻量级，不需要复杂的安装手段。

* 配置简单，与`gitlab`可直接适配。
* 实时构建日志十分清晰，`UI`交互体验很好
* 使用 `YAML` 进行配置，任何人都可以很方便的使用。

### gitlab-CI劣势

* 没有统一的管理界面，无法统筹管理所有项目
* 配置依赖于代码仓库，耦合度没有`Jenkins`低

### Jenkins优势

* 编译服务和代码仓库分离，耦合度低
* 插件丰富，支持语言众多。
* 有统一的`web`管理界面

### Jenkins劣势

* 插件以及自身安装较为复杂
* 对`docker`插件支持不是很友好
* 体量较大，不是很适合小型团队

最后我选择了`gitlab-CI`，因为我们团队使用的`gitlab`，和`gitlab-CI`无缝连接，由于我们使用了`docker`，所以在选型`jenkins`的时候，感觉插件有点难用。

## 总结

我认为选择工具本身就是一个“持续迭代”的过程，这个过程中根据自己团队的特点和规模，选择最合适目前的工具就好了。等到工具已经没法满足团队需求了，再选择更加合适的。
