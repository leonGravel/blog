---
title: k8s集群搭建
categories: Docker
tags: [docker,k8s]
date: 2018-09-10 20:54:41
---

### 使用minikube单机部署
[minikube](https://github.com/kubernetes/minikube)是一个用go语言开发的可以在本地运行kubernetes的利器。首先，我们需要安装它：

<!--more-->

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
```
然后检查一下
```
minikube version
```
如果失败，则重复上一步
然后启动minikube：
```
minikube start
```
#### 安装Virtualbox
如果你的机子没有安装virtualBox，会报错。`Error starting host:  Error creating. Error with pre-create check: "VBoxManage not found. Make sure VirtualBox is installed and VBoxManage is in the path"`
我的机子是centos:
```
cd /etc/yum.repos.d
wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

## 更新缓存
yum clean all
yum makecache
## 安装virtualbox
yum install VirtualBox-5.1
```
安装完成之后，再次启动；
如果出现这种错误
```
minikube start
Starting local Kubernetes cluster...
E0727 06:41:54.512097    3933 start.go:78] Error starting host:  Error creating. Error with pre-create check: "We support Virtualbox starting with version 5. Your VirtualBox install is \"WARNING: The vboxdrv kernel module is not loaded. Either there is no module\\n         available for the current kernel (3.10.0-327.22.2.el7.x86_64) or it failed to\\n         load. Please recompile the kernel module and install it by\\n\\n           sudo /sbin/vboxconfig\\n\\n         You will not be able to start VMs until this problem is fixed.\\n5.1.2r108956\". Please upgrade at https://www.virtualbox.org"
```
执行一下脚本，启动Virtualbox的Service
```
/usr/lib/virtualbox/vboxdrv.sh setup
```
启动