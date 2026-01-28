---
title: Spring Boot 2.x 编写 RESTful API (六) 事务
date: 2019-04-08 17:00:00
updated: 2019-04-08 17:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记


# Transactional

![](https://oss.x8y.cc/blog-img/2060/6/transactional.PNG)

# 判定顺序

![](https://oss.x8y.cc/blog-img/2060/6/commit-rollback.PNG)

# propagation

![](https://oss.x8y.cc/blog-img/2060/6/propagation.PNG)

# isolation

![](https://oss.x8y.cc/blog-img/2060/6/isolation.PNG)

## 脏读

![](https://oss.x8y.cc/blog-img/2060/6/dirty-read.PNG)

## 不可重复读

![](https://oss.x8y.cc/blog-img/2060/6/non-repeatable-read.PNG)

## 幻读

![](https://oss.x8y.cc/blog-img/2060/6/phantom-read.PNG)

不可重复读是指记录不同 (update)，幻读是数据条数不同 (insert, delete)

## 几种隔离的比较

![](https://oss.x8y.cc/blog-img/2060/6/isolation-compare.PNG)

## isolation vs. lock

+ 两个不同的东西，隔离不是靠锁实现的，是靠对数据的监控实现的
+ 锁：表加好锁了，除非出现死锁等特殊状况，事务是不会被数据库主动回滚的
+ 隔离：如果发现数据不符合相应的事务隔离级别，当前事务会出错并回滚；相比锁被回滚可能较大，需要程序有出错重试的步骤

# timeout

+ timeout 事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务
+ 方法抛出异常，事务被回滚，可能是 SQL 执行时间过长的异常，也可能是 TransactionTimedOutException
+ 从方法执行开始计算，每个 SQL 执行前检查一次是否超时，方法全部执行完毕后不检查是否超时

 
源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)