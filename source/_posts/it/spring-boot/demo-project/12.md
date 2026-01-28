---
title: Spring Boot 构建电商基础秒杀项目 (十二) 总结 (完结)
date: 2019-03-24 17:00:00
updated: 2019-03-24 17:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# 系统架构

![](https://oss.x8y.cc/blog-img/2047/architecture.png)

# 存在问题

+ 如何发现容量问题
+ 如何使得系统水平扩展
+ 查询效率低下
+ 活动开始前页面被疯狂刷新
+ 库存行锁问题
+ 下单操作步骤多，缓慢
+ 浪涌流量如何解决

源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

