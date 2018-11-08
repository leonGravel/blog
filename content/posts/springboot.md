---
title: springboot
categories: springboot
tags: [java,springboot]
author: liao

date: 2018-07-11 23:01:41 
---


## Spring Boot 简介
Spring Boot(英文中是`引导`的意思)，是用来简化Spring应用的搭建到开发的过程。应用开箱即用，只要通过 `just run`（可能是 java -jar 或 tomcat 或 maven插件run 或 shell脚本），就可以启动项目。二者，Spring Boot 只要很少的Spring配置文件（例如那些xml，property）。 因为`习惯优先于配置`的原则，使得Spring Boot在快速开发应用和微服务架构实践中得到广泛应用。   Javaer装好JDK环境和Maven工具就可以开始学习Boot了~

<!--more-->

## Hello world
首先得有个maven基础项目，可以直接使用Maven骨架工程生成Maven骨架Web项目，即man archetype:generate命令：
```
mvn archetype:generate -DgroupId=springboot -DartifactId=springboot-helloworld -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
### pom.xml配置
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>springboot</groupId>
    <artifactId>springboot-helloworld</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-helloworld :: HelloWorld Demo</name>

    <!-- Spring Boot 启动父依赖 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.3.RELEASE</version>
    </parent>

    <dependencies>
        <!-- Spring Boot web依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
</project>
```
只要加入一个 Spring Boot 启动父依赖即可。  
### Controller层
HelloWorldController的代码如下：
```
/**
 * Spring Boot HelloWorld案例
 *
 * Created by liao on 2018/7/12.
 */
@RestController
public class HelloWorldController {

    @RequestMapping("/")
    public String sayHello() {
        return "Hello,World!";
    }
}
```
@RestController和@RequestMapping注解是来自SpringMVC的注解，它们不是SpringBoot的特定部分。 1. @RestController：提供实现了REST API，可以服务JSON,XML或者其他。这里是以String的形式渲染出结果。 2. @RequestMapping：提供路由信息，"/"路径的HTTP Request都会被映射到sayHello方法进行处理。 
### 启动应用类
和第一段描述一样，开箱即用。如下面Application类：
```
/**
 * Spring Boot应用启动类
 *
 * Created by liao on 2018/7/12.
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```
1. @SpringBootApplication：Spring Boot 应用的标识 
2. Application很简单，一个main函数作为主入口。SpringApplication引导应用，并将Application本身作为参数传递给run方法。具体run方法会启动嵌入式的Tomcat并初始化Spring环境及其各Spring组件。  

## 运行
Just Run的宗旨，运行很简单，直接右键Run运行Application类。同样你也可以Debug Run。可以在控制台中看到：
```
Tomcat started on port(s): 8080 (http)
Started Application in 5.986 seconds (JVM running for 7.398)
```
然后访问[http://localhost:8080/][1],即可在页面中看到Spring Boot对你 say hello：
```
Hello,World！
```
## 配置文件
我在Spring Boot使用过程中，最直观的感受就是没有了原来自己整合Spring应用时繁多的XML配置内容，替代它的是在pom.xml中引入模块化的Starter POMs，其中各个模块都有自己的默认配置，所以如果不是特殊应用场景，就只需要在application.properties中完成一些属性配置就能开启各模块的应用。
### 自定义属性与加载
在使用Spring Boot的时候，通常也需要定义一些自己使用的属性，可以如下方式直接定义：
```
com.gravel.name=李鳌
com.gravel.title=李鳌的文章
```
然后通过`@Value("${属性名}")`注解来加载对应的配置属性，具体如下：
```
@Component
public class gravelProperties {

    @Value("${com.gravel.name}")
    private String name;
    @Value("${com.gravel.title}")
    private String title;

    // 省略getter和setter

}
```
可以通过单元测试来验证`gravelProperties`中的属性是否已经根据配置文件加载了。
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
public class ApplicationTests {

    @Autowired
    private gravelProperties tProperties;


    @Test
    public void getHello() throws Exception {
        Assert.assertEquals(tProperties.getName(), "李鳌");
        Assert.assertEquals(tProperties.getTitle(), "李鳌的文章");
    }

}
```
### 属性配置类
springboot还支持引入外部配置文件，这里举个例子。属性配置类 `StaticProperties.class`
```
@Component
public class StaticProperties {

    public static String CUSTOM_NAME;

    @Value("${custom.name}")
    public void setCustomName(String customName) {
        CUSTOM_NAME = customName;
    }

}
```
Spring Boot 配置提示 resources/META-INF/spring-configuration-metadata.json
```
{
  "properties": [
    {
      "name": "custom.name",
      "type": "java.lang.String",
      "sourceType": "com.gravel.config.StaticProperties"
    }
  ]
}
```
Spring Boot 配置 application.properties
```
custom.name=liao
```
至此，即可在 Spring Boot 全局任意引用 StaticProperties.CUSTOM_NAME来获取这个属性啦。
### 参数间的引用
在`application.properties`中的各个参数之间也可以直接引用来使用，就像下面的设置：
```
com.gravel.name=李鳌
com.gravel.title=李鳌的文章
com.gravel.blog.desc=${com.gravel.name}正在努力写《${com.gravel.title}》
```
`com.gravel.blog.desc`参数引用了上文中定义的name和title属性，最后该属性的值就是李鳌正在努力写《李鳌的文章》。

### 通过命令行设置属性值
相信使用过一段时间`Spring Boot`的用户，一定知道这条命令：`java -jar xxx.jar --server.port=8888`，通过使用`—server.port`属性来设置`xxx.jar`应用的端口为`8888`。

在命令行运行时，连续的两个减号--就是对`application.properties`中的属性值进行赋值的标识。所以，`java -jar xxx.jar --server.port=8888`命令，等价于我们在`application.properties`中添加属性`server.port=8888`，该设置在样例工程中可见，读者可通过删除该值或使用命令行来设置该值来验证。

通过命令行来修改属性值固然提供了不错的便利性，但是通过命令行就能更改应用运行的参数，那岂不是很不安全？是的，所以Spring Boot也贴心的提供了屏蔽命令行访问属性的设置，只需要这句设置就能屏蔽：`SpringApplication.setAddCommandLineProperties(false)`。
### 多环境配置
我们在开发Spring Boot应用时，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：
* `application-dev.properties`：开发环境
* `application-test.properties`：测试环境
* `application-prod.properties`：生产环境

至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。
如：`spring.profiles.active=test`就会加载`application-test.properties`配置文件内容。`
下面，以不同环境配置不同的服务端口为例，进行样例实验。

针对各环境新建不同的配置文件`application-dev.properties`、`application-test.properties`、`application-prod.properties`;

在这三个文件均都设置不同的`server.port`属性，如：dev环境设置为1111，test环境设置为2222，prod环境设置为3333,

`application.properties`中设置`spring.profiles.active=dev`，就是说默认以dev环境设置

测试不同配置的加载

* 执行`java -jar xxx.jar`，可以观察到服务端口被设置为1111，也就是默认的开发环境（dev）
* 执行`java -jar xxx.jar --spring.profiles.active=test`，可以观察到服务端口被设置为2222，也就是测试环境的配置（test）
* 执行`java -jar xxx.jar --spring.profiles.active=prod`，可以观察到服务端口被设置为3333，也就是生产环境的配置（prod）
按照上面的实验，可以如下总结多环境的配置思路：

`application.properties`中配置通用内容，并设置`spring.profiles.active=dev`，以开发环境为默认配置
`application-{profile}.properties`中配置各个环境不同的内容
通过命令行方式去激活不同环境的配置。

[1]: http://localhost:8080/
