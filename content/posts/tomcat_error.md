---
title: tomcat启动报错
categories: java 
tags: [tomcat,bug] 
aothor : gravel 
date: 2018-04-04 20:54:41 
---

今天启动项目的时候，tomcat报了
```
are only available on JDK 1.5 and higher
```
的错。查了一些资料发现，这是因为jdk升级成为了jdk8，但是spring的jar包版本比较低，并没有兼容到jdk8，所以才造成了现在的这个错误。

<!--more-->

想要解决这个错误，一般有两种办法：

1. 将jdk8换成jdk7，重新启动项目就好了。

2. 第二种手动修改spring的jar包，在org.springframework.core目录下，有一个JdkVersion.class，自己参照包路径，重新写一个JdkVersion.java，如下所示：。<!-- more -->

```
package org.springframework.core;  
  
public class JdkVersion  
{  
      
    public static final int JAVA_13 = 0;  
      
    public static final int JAVA_14 = 1;  
      
    public static final int JAVA_15 = 2;  
      
    public static final int JAVA_16 = 3;  
      
    public static final int JAVA_17 = 4;  
      
    //for jre 1.8  
    public static final int JAVA_18 = 5;  
      
    private static final String javaVersion = System.getProperty("java.version");  
      
    private static final int majorJavaVersion;  
      
    public static String getJavaVersion()  
    {  
        return javaVersion;  
    }  
      
    public static int getMajorJavaVersion()  
    {  
        return majorJavaVersion;  
    }  
      
    public static boolean isAtLeastJava14()  
    {  
        return true;  
    }  
      
    public static boolean isAtLeastJava15()  
    {  
        return getMajorJavaVersion() >= 2;  
    }  
      
    public static boolean isAtLeastJava16()  
    {  
        return getMajorJavaVersion() >= 3;  
    }  
      
    static  
    {  
        //for jre 1.8  
        if (javaVersion.indexOf("1.8.") != -1)  
        {  
            majorJavaVersion = 5;  
        }  
        else if (javaVersion.indexOf("1.7.") != -1)  
        {  
            majorJavaVersion = 4;  
        }  
        else if (javaVersion.indexOf("1.6.") != -1)  
        {  
            majorJavaVersion = 3;  
        }  
        else if (javaVersion.indexOf("1.5.") != -1)  
        {  
            majorJavaVersion = 2;  
        }  
        else  
        {  
            majorJavaVersion = 1;  
        }  
    }  
} 
```
写好之后，编译成.class文件，放到spring的jar包中，替换项目jar包，重新启动，就好了。