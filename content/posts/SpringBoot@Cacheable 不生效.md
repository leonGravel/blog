---
title: Springboot@Cacheable 不生效
categories: ["Java基础"]
tags: ["springboot","guava","cache"]
date: 2019-07-16 20:54:41
---

最近在项目中使用了Guava缓存，使用方式是用Spring提供的 @Cacheable 注解的方式，在使用的过程中，遇到了缓存不生效的情况。

Spring 使用@Cacheable添加缓存是基于面向切面的思想做的，实际上就是使用Java动态代理，创建实例的时候注入的是代理对象，在代理对象里调用实际的对象，这样就可以在实际的方法执行前，处理一下缓存的逻辑：没有找到缓存就往下执行，执行完把结果加入到缓存中；找到缓存则直接返回缓存的结果，不调用执行实际的方法。

解决方案

1. 不使用注解的方式，直接取 Ehcache 的 CacheManger 对象，把需要缓存的数据放到里面，类似于使用 Map，缓存的逻辑自己控制；或者可以使用redis的缓存方式去添加缓存；

2. 把方法A和方法B放到两个不同的类里面，例如：如果两个方法都在同一个service接口里，把方法B放到另一个service里面，

3. 在guava缓存配置类上，加上@EnableAspectJAutoProxy(exposeproxy = true)，这个注解的作用是可以允许同类中的方法互相切入。然后通过AOP切入同类调用方法-AopContext.currentProxy()，即可通过代理调用@Cacheable 注解的方法。这样就能走代理了

