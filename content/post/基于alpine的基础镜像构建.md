---
title: docker安装redis并使用外部配置和数据初始化
categories: ["Docker"]
tags: ["docker","redis"]
date: 2018-09-19 23:01:41 
author: gravel

---

刚开始在项目中使用docker的时候，使用的是centos作为基础镜像。centos的官方镜像有70M左右。加上jdk、tomcat，一个完整的业务系统，可能有450M左右。在项目组同学去试用部署的时候，所以的应用包加上服务包，有点过于大了。而且以centos作为基础镜像，可能包含很多我们并不需要的功能。所以，决定转用alpine。

<!--more-->

### alpine介绍

在alpine的官网上写着：

```
Alpine Linux is a security-oriented, lightweight Linux distribution based on musl libc and busybox.//Alpine Linux 是一个社区开发的面向安全应用的轻量级 Linux 发行版。
```

最令人吃惊的是，他的官方docker镜像，只有5M。所以在2016年的时候，docker官方已经将推荐基础镜像，从Ubuntu转为alpine。

### 编写基础镜像的Dockerfile

alpine优点是安全，体积小，但同时，他肯定也缺少一些本地化的东西。

我尝试过直接将alpine作为基础镜像去运行一个tomcat应用，发现在解压war包的时候，里面的中文文件无法被识别。另外无法直接将时区设置为东八区。

在结合网上资料整理之后发现，alpine主要也是这两个问题困扰着使用中文的程序员。所以我们需要定制化自己的alpine基础镜像。结合Dockerfile来说吧。

```
FROM  alpine:3.6

MAINTAINER leongravel "leebroncc@gmail.com"

## 设置默认语言环境
ENV LANG=C.UTF-8

# 安装 GNU libc (aka glibc)和C.UTF-8 locale的依赖 以及设置时区
# 下面这么长一串，主要是通过apk安装glibc的依赖，他的作用主要是本地化支持，和字符集的切换。
RUN ALPINE_GLIBC_BASE_URL="https://github.com/sgerrand/alpine-pkg-glibc/releases/download" && \
    ALPINE_GLIBC_PACKAGE_VERSION="2.27-r0" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    apk add --no-cache --virtual=.build-dependencies wget ca-certificates && \
    echo \
        "-----BEGIN PUBLIC KEY-----\
        MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApZ2u1KJKUu/fW4A25y9m\
        y70AGEa/J3Wi5ibNVGNn1gT1r0VfgeWd0pUybS4UmcHdiNzxJPgoWQhV2SSW1JYu\
        tOqKZF5QSN6X937PTUpNBjUvLtTQ1ve1fp39uf/lEXPpFpOPL88LKnDBgbh7wkCp\
        m2KzLVGChf83MS0ShL6G9EQIAUxLm99VpgRjwqTQ/KfzGtpke1wqws4au0Ab4qPY\
        KXvMLSPLUp7cfulWvhmZSegr5AdhNw5KNizPqCJT8ZrGvgHypXyiFvvAH5YRtSsc\
        Zvo9GI2e2MaZyo9/lvb+LbLEJZKEQckqRj4P26gmASrZEPStwc+yqy1ShHLA0j6m\
        1QIDAQAB\
        -----END PUBLIC KEY-----" | sed 's/   */\n/g' > "/etc/apk/keys/sgerrand.rsa.pub" && \
    wget \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    rm "/etc/apk/keys/sgerrand.rsa.pub" && \
    /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 "$LANG" || true && \
    echo "export LANG=$LANG" > /etc/profile.d/locale.sh && \
    \
    apk del glibc-i18n && \
    \
    rm "/root/.wget-hsts" && \
    apk del .build-dependencies && \
    rm \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
  echo 'http://mirrors.ustc.edu.cn/alpine/v3.6/main' > /etc/apk/repositories \
    && echo 'http://mirrors.ustc.edu.cn/alpine/v3.6/community' >>/etc/apk/repositories \
&& apk update && apk add tzdata \
&& ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \ 
&& echo "Asia/Shanghai" > /etc/timezone

## 上面这三行代码，是通过apk添加tzdata支持，将时区设置为东八区，也就是中国区的时间。
```

构建docker镜像

```
docker build -t base-alpine . // 构建镜像
```

通过以上这个命令，我们就完成了基础镜像的构建。这个镜像只比最纯净的alpine官方镜像大2M。

### 通过基础镜像构建其他应用

完成以上的基础镜像之后，其实大部分工作都已经完成了。下面也通过Dockerfile来讲个例子。

1. jdk8的镜像构建

```
FROM cdjd/ci/base-alpine

MAINTAINER  leongravel "leebroncc@gmail.com"

ENV JAVA_VERSION=8 \
    JAVA_UPDATE=192 \
    JAVA_BUILD=12 \
    JAVA_PATH=750e1c8617c5452694857ad95c3ee230 \
    JAVA_HOME="/usr/lib/jvm/default-jvm"

RUN apk add --no-cache --virtual=build-dependencies wget ca-certificates unzip && \
    cd "/tmp" && \
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
        "http://download.oracle.com/otn-pub/java/jdk/${JAVA_VERSION}u${JAVA_UPDATE}-b${JAVA_BUILD}/${JAVA_PATH}/jdk-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz" && \
    tar -xzf "jdk-${JAVA_VERSION}u${JAVA_UPDATE}-linux-x64.tar.gz" && \
    mkdir -p "/usr/lib/jvm" && \
    mv "/tmp/jdk1.${JAVA_VERSION}.0_${JAVA_UPDATE}" "/usr/lib/jvm/java-${JAVA_VERSION}-oracle" && \
    ln -s "java-${JAVA_VERSION}-oracle" "$JAVA_HOME" && \
    ln -s "$JAVA_HOME/bin/"* "/usr/bin/" && \
    rm -rf "$JAVA_HOME/"*src.zip && \
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie;" \
        "http://download.oracle.com/otn-pub/java/jce/${JAVA_VERSION}/jce_policy-${JAVA_VERSION}.zip" && \
    unzip -jo -d "${JAVA_HOME}/jre/lib/security" "jce_policy-${JAVA_VERSION}.zip" && \
    rm "${JAVA_HOME}/jre/lib/security/README.txt" && \
    apk del build-dependencies && \
    rm "/tmp/"*
```

这个dockerfile比上面的那个要好理解一点，首先基于我们已经构建好的cdjd/ci/base-alpine，从oracle官网上，通过wget下载JDK8，然后解压。将解压后的文件移动到定义好的java-home,然后同ln软连接，配置java环境变量。然后下载jce_policy补丁包，放入/jre/lib/security文件夹。 删除一些不必要文件。

然后：

```
docker build -t jdk1.8 . // 构建镜像
```

完成JDK8环境的构建。我们可以通过这个镜像，去运行一些springboot的项目。

接下来说下tomcat镜像，tomcat我这里选用的是tomcat7，Dockerfile如下：

```
# 版本信息
FROM jdk1.8
MAINTAINER  leongravel "leebroncc@gmail.com"

# add tomcat
COPY tomcat /opt/tomcat

#设置环境变量
ENV TOMCAT_HOME /opt/tomcat \
  PATH ${PATH}:${TOMCAT_HOME}/bin

#开启内部服务端口
EXPOSE 8080
ENTRYPOINT ["/opt/tomcat/bin/catalina.sh", "run"]
```

这里我是把已经解压好的tomcat copy进镜像构建，然后设置tomcat环境变量。



### Dockerfile相关问题解释

我们知道镜像都是由一层一层的layer构成的，而这一层一层的layer就是我们所写的dockerfile里面的命令所定义的，比如:

```
RUN mkdir -p /opt/thunisoft
RUN cd /opt/thunisoft
```

这就会生成两个layer，所以我们再编写dockerfile时候，应该尽可能的将所有的命令，都通过各种方式合并起来，这样构建的镜像会比两条命令，小很多。

```
RUN mkdir -p /opt/thunisoft && \
    cd /opt/thunisoft 
```

以上是正确示范。

另外，如果需要在dockerfile里面下载外部资源，最好下载到tmp目录里面，使用完成之后，再将这个目录统一删除。