---
title: Spring Boot 2.x 编写 RESTful API (一) RESTful API 介绍 & RestController
date: 2019-04-04 07:00:00
updated: 2019-04-04 07:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记

# RESTful API 介绍

+ REST 是 Representational State Transfer 的缩写
+ 所有的东西都是资源，所有操作都通过对资源的增删改查 (CRUD) 实现
+ 对资源的增删改查对应对 URL 的操作 (POST, DELETE, PUT, GET)
+ 无状态的

URL 中都应该是名词，不应该出现动词


关于 RESTful API 更详细可参考阮一峰老师的 [RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

# RestController

![](https://oss.x8y.cc/blog-img/2060/1/RestController.jpg)


源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)