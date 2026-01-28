---
title: 学习 Spring (十二) AOP 基本概念及特点
date: 2019-03-04 7:00:00
updated: 2019-03-04 7:00:00
categories: [IT]
tags: [Java, Spring]
---

> [Spring入门篇](https://www.imooc.com/learn/196) 学习笔记

AOP: Aspect Oriented Programming, 通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术

主要功能是：日志记录、性能统计、安全控制、事务处理、异常处理等

# AOP 实现方式

1. 预编译：AspectJ
1. 运行期动态代理(JDK 动态代理, CGLib 动态代理)：SpringAOP, JbossAOP

# AOP 相关概念

+ 切面 (Aspect)：一个关注点的模块化，这个关注点可能会横切多个对象
+ 连接点 (Joinpoint)：程序执行过程中某个特定的点
+ 通知 (Advice)：在切面的某个特定的连接点上执行的动作
+ 切入点 (Pointcut)：匹配连接点的断言，在 AOP 中通知和一个切入点表达式关联
+ 引入 (Introduction)：在不修改类代码的前提下，为类添加新的方法和属性
+ 目标对象 (Target Object)：被一个或者多个切面所通知的对象
+ AOP 代理 (AOP Proxy)：AOP 框架创建的对象，用来实现切面契约(aspect contract)(包括通知方法执行等功能)
+ 织入 (Weaving)：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象，分为：编译时织入、类加载时织入、执行时织入


# Advice 的类型

1. 前置通知 (Before advice)：在某连接点之前执行的通知，但不能阻止连接点前的执行(除非它抛出一个异常)
1. 返回后通知 (After returning advice)：在某连接点正常完成后执行的通知
1. 抛出异常通知 (After throwing advice)：在方法抛出异常退出时执行的通知
1. 后通知 (After (finally) advice)：当某连接点退出的时候执行的通知(不论是正常返回还是异常退出)
1. 环绕通知 (Around advice)：包围一个连接点的通知

# Spring 框架中 AOP 的用途

+ 提供了声明式的企业服务，特别是 EJB 的替代服务的声明
+ 允许用户定制自己的切面，以完成 OOP 与 AOP 的互补使用

# Spring 的 AOP 实现方式：

+ 纯 java 实现，无需特殊大的编译过程，不需要控制类加载器层次
+ 目前只支持方法执行连接点(通知 Spring Bean 的方法执行)
+ 不是为了提供最完整的 AOP 实现，而是侧重于提供一种 AOP 实现和 Spring IoC 容器之间的整合，用于帮助解决企业应用中的常见问题
+ Spring AOP 不会与 AspectJ 竞争，不会提供综合全面的 AOP 解决方案

# 有接口和无接口的 Spring AOP 实现区别

+ Spring AOP 默认使用标准的 Jave SE 动态代理作为 AOP 代理，这使得任何接口(或者接口集)都可以被代理
+ Spring AOP 中也可以使用 CGLIB 代理(如果一个业务对象并没有实现一个接口)



# 源码：[learning-spring](https://github.com/VictorBu/learning-spring)
