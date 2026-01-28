---
title: Spring Boot 2.x 编写 RESTful API (三) 程序层次 & 数据传输
date: 2019-04-06 19:00:00
updated: 2019-04-06 19:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记


# 程序的层次结构

![](https://oss.x8y.cc/blog-img/2060/3/architecture.PNG)

![](https://oss.x8y.cc/blog-img/2060/3/pbf-pbl.PNG)


# 相邻层级的数据传输

![](https://oss.x8y.cc/blog-img/2060/3/xo.PNG)

![](https://oss.x8y.cc/blog-img/2060/3/transfer.PNG)


# JavaBean

+ 有一个 public 的无参构造方法
+ 属性 private，且可以通过 get、set、is (可以替代 get，用在布尔属性上) 方法或遵循特定命名规范的其他方法访问
+ 可序列化，实现 Serializable 接口

# POJO & JavaBean

+ POJO 比 JavaBean 更简单。POJO 严格地遵守简单对象的概念，而一些 JavaBean 中往往会封装一些简单逻辑
+ POJO 主要用于数据的临时传递，它只能装载数据，作为数据存储的载体，不具有业务逻辑处理的能力
+ JavaBean 虽然数据的获取与 POJO 一样，但是 JavaBean 当中可以有其他的方法

# 几种简化方案？

+ 一种 POJO 从 web 控制层到数据访问层 (小项目可以使用)
+ 用 JavaBean 代替 POJO (不建议使用)
+ POJO 的 get/set 写起来也麻烦，用 public 的 field 代替 (不建议使用)


源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)