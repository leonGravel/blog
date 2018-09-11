---
title: gitlab runner + docker 自动构建
categories: Docker
tags: [docker,gitlab,CI]
date: 2018-9-11 23:01:41 
author: gravel
---

最开始的时候，我尝试Jenkins+docker，可是Jenkins的docker依赖和插件，实在太过麻烦，配置项等等，太重。所以我转为使用gitlab runner来实现自动构建并打包镜像。
## 准备工作
* 安装docker
* 在docker同一个机器上安装gitlab runner
* 配置.gitlab-ci.yml

### 安装Docker 
参考官方文档，唯一需要注意的是，需要将镜像仓库地址修改为私有的地址。可以通过配置Deamon.json实现。具体配置如下。
`registry-mirrors`代表私有仓库地址，`insecure-registries`的作用是，push指向的地址。
> Docker自从1.3.X之后docker registry交互默认使用的是HTTPS，所以这里最好直接指定仓库地址，不然push的时候会报错。


```
{
  "registry-mirrors": [
    "http://registry.******.com:5000"
  ],
  "insecure-registries": [
    "registry.******.com:5000"
  ],
  "debug": true,
  "experimental": false
}
```
### 安装gitlab runner
服务器是centos 7,以下的步骤都是基于centos 7
#### 下载二进制安装文件
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

#### 安装
```
sudo yum install gitlab-runner
```

#### 注册Runner
```
sudo gitlab-runner register
```

1. 填入填入私有gitlab的url
```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.******.com
```

2. 输入项目的gitlab token
```
Please enter the gitlab-ci token for this runner
xxx
```

3. 添加runner描述
```
Please enter the gitlab-ci description for this runner
jiedu-ci
```
4. 添加描述标签，若添加多个需用逗号隔开
```
Please enter the gitlab-ci tags for this runner (comma separated):
ci,shws,jiedu
```

5. 选择runner 运行模式
这里我选择的Docker，因为我们要使用docker 镜像以及一些其他的镜像

>其他的我也不是很了解

```
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker
```
6. 设置基础镜像
```
Please enter the Docker image (eg. ruby:2.1):
registry.******.com:5000/dzjz/ci/maven
```

#### 配置gitlab runner
```
vi /etc/gitlab-runner/config.toml
```
写入
```
concurrent = 1
check_interval = 0
environment = ["MAVEN_HOME=/path/to/maven"]
[[runners]]
  name = "shws-ci"
  url = "http://gitlab.******.com"
  token = "234234*******************"
  executor = "docker"
  output_limit = 208192
  [runners.docker]
    tls_verify = false
    image = "registry.******.com:5000/dzjz/ci/maven"
    privileged = true
	cache_dir = "cache"
    disable_cache = false
     volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache","/root/.m2:/root/.m2"]
    shm_size = 0
	pull_policy = "if-not-present"
  [runners.cache]

```
当然这个部分后期会进一步完善修改，目前只是一个示例。

### 配置.gitlab-ci.yml
这一部分，具体可以参考官方文档以及[这篇](https://segmentfault.com/a/1190000011890710)，项目的配置如下：
```
# 定义stages
stages:
  - build
  - push
  


# 构建各个依赖组件的jar包，并复制Dockerfile对应位置等待构建.
build-job:
  #  image: registry.******.com:5000/dzjz/ci/maven
    stage: build
    only:
        - feature/20180428-1.0.0.0script1-第二版

    script:
        - mkdir -p jd-ci/shws/kf
        - cd shws/ && pwd
        - mvn clean install -U
        - echo '准备发布生活卫生镜像到私有镜像仓库！'
        - rm -rf src/dockerfile/*.war
        - cd .. && pwd
        - cp -r shws/target/shws.war shws/src/dockerfile/Dockerfile jd-ci/shws/kf
        - cd jd-ci/shws/kf/ && ls
    ## 下面这个配置的作用是在不同的job间传递共享war包以及Dockerfile
    artifacts:
      name:  "${CI_REGISTRY_IMAGE}:${CI_COMMIT_REF_SLUG}"
      paths:
        - jd-ci/shws/kf/*
      expire_in: 1 day


push-job:
  stage: push
  only:
    - feature/20180428-1.0.0.0script1-第二版
  image: registry.******.com:5000/docker:latest
  services: 
    - registry.******.com:5000/docker:dind
  before_script:
  - docker info
  script:
    - echo '准备构建镜像并push到私有仓库'
    #- cd ./jd/shws/kf/ && ls
    - docker stop shws-kf || true
    - docker rm -f shws-kf || true
    - docker rmi registry.******.com:5000/cdjd/shws-kf || true
    - docker build -t registry.******.com:5000/cdjd/shws-kf ./jd-ci/shws/kf/
    - docker push registry.******.com:5000/cdjd/shws-kf
    - rm -rf jd-ci/shws/kf

```

这个配置文件中，`only`部分代表你的分支，script部分代表对应的脚本。其他业务系统需要替换的东西（比如构建镜像的名字，缓存的名称，这些可以自定义）。
### 结束
以上就是gitlab runner + docker实现自动构建打包镜像并上传的简单教程。