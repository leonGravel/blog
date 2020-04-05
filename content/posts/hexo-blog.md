---
title: HEXO+Github,搭建个人博客
date: 2016-04-22 19:33:40 #发表日期，一般不改动
categories: ["博客"]#文章文类
tags: ["hexo"] #文章标签，不同主题对于不同的标签的分类不太一样，这一点我正在研究

---
很久之前就想搭建一个自己的个人博客，甚至想过自己的毕业设计的题目就选择这个。但因为种种原因(`懒`)，直到上个星期才通过hexo完成博客的搭建。为了方便大家`(装逼`)也不至于让我这里变得冷清，我决定还是写一个简单平快的教程出来。文笔拙劣，还请赐教。

<!--more-->

## 正文
由于我没有mac，所以本教程只针对windows用户哟。

## 配置环境
### 安装git
Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。Git 是大神Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。在这里我们主要用它来实现把本地的hexo内容提交到github也就是我们的博客上去。你可以选择[git官网](http://git-scm.com/download/win/)或者[git国内下载站](https://github.com/waylau/git-for-win/)下载

### 安装Node.js
由于hexo是基于Node.js的博客程序，所以安装[Node.js](http://nodejs.cn/)是必须的。可以参考此教程来安装。

### 安装hexo
完成以上步骤之后，可以开始我们的hexo安装。在此我建议新建一个文件夹blog来存放关于博客相关的内容和hexo装文件。在这个文件夹下右键git bash，用以下命令安装hexo.
```
$ npm install hexo-cli -g
```
在blog文件夹下面创建hexo文件夹，在这使用以下命令初始化博客搭建所需的所有文件
```
$ hexo init
```
安装依赖包
```
$ npm install
```
在当前文件夹执行以下命令
```
$ hexo g
$ hexo s
```

如果一切顺利的话，在浏览器输入`http://localhost:4000/`,就可以看到hexo生成的本地博客，hexo的默认主题是landscape，知乎上有人总结hexo的许多好看的主题，有兴趣的可以下载替换。

### 注册Github
如果你已经有[github](https://github.com/)账号，可以跳过此步骤。
### 创建repository
repository是你的个人仓库，你可以往里面放你的项目代码，在这里我们新建一个repository。如图所示
![github示例图][1]

  红框处的项目名必须为`username.github.io`这种格式，如我的github用户名为L-Gravel,那么这个项目名就是L-Gravel.github.io,这样可以方便github解析你的博客地址。之后点击箭头所指的按钮生成repository.
  ## 连接本地git与github
  ### 配置Git\

  首先在本地创建ssh key；
  ```
  ssh-keygen -t rsa -C "your_email@youremail.com"
  ```
后面的your_email@youremail.com改为你的邮箱，之后会要求确认路径和输入密码，我们这使用默认的一路回车就行。成功的话会在~/下生成.ssh文件夹，进去，打开id_rsa.pub，复制里面的key。

回到github，进入Settings
![enter description here][2]

左边选择SSH Keys，new SSH Key,title随便填，粘贴key。为了验证是否成功，右键git bash输入：
```
$ ssh -T git@github.com
```
如果是第一次的会提示是否`continue`，输入yes就会看到：`You’ve successfully authenticated, but GitHub does not provide shell access`。这就表示已成功连上github。
接下来我们要做的就是把本地仓库传到github上去，在此之前还需要设置username和email，因为github每次commit都会记录他们。
```
$ git config --global user.name "your name"  
$ git config --global user.email "your_email@youremail.com"
```
在这里填写你们的github邮箱就好。
### 将本地博客文件上传到github
在上一步，我们创建了博客代码所需的仓库，并且建立了本地git和github的链接。现在当然是要把本地的博客代码上传到仓库里面。编辑hexo文件夹路径下的`_config.yml`文件。在这个文件末尾添加以下代码：
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy: 
  type: git
  repository: git@github.com:L-Gravel/L-Gravel.github.io.git
  branch: master
```
  注意，`L-Gravel`应该替换为你自己的用户名。配置好`_config.yml`并保存后，执行以下命令部署到Github上。
  ```
  $ hexo g
$ hexo d
  ```
如果这个时候出现`ERROR Deployer not found : github`,可以试试输入以下代码：
```
$ npm install hexo-deployer-git --save
$ hexo g
$ hexo d
```
执行完以上步骤之后，如果一切顺利的话，博客就应该已经顺利的发布在github上了，可以通过`username.github.io`直接访问。
## 发布一篇文章
1.在Git Bash执行命令：
```
$ hexo new "my new post"
```
2.在hexo\source_post中打开my-new-post.md，建议使用notepad++或editplus。
hexo中写文章使用的是Markdown，没接触过的可以看下[Markdown相关教程](http://sspai.com/25137)。
```
title: my new post #可以改成中文或者其他的，如“第一篇博客”
date: 2016-04-22 19:33:40 #发表日期，一般不改动
categories: blog #文章文类
tags: [博客，文章] #文章标签，不同主题对于不同的标签的分类不太一样，这一点我正在研究
---
#这里是正文，用markdown写，你可以选择写一段显示在首页的简介后，加上
<!--more-->，在<!--more-->之前的内容会显示在首页，之后的内容会被隐藏，当游客点击Read more才能看到。
```
## 总结
至此为止，我们已经完成了基础hexo的博客搭建，关于域名绑定这一块我之后会写另外一篇博客来说明。如果你参考此文遇到了什么问题，欢迎点击右边图标链接或是在下方留言与我联系。

[1]: http://7xt2ta.com2.z0.glb.clouddn.com/1.png
[2]: http://7xt2ta.com2.z0.glb.clouddn.com/2.png