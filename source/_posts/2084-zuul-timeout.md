---
title: Zuul 超时设置
date: 2019-07-15 19:30:00
updated: 2019-07-15 19:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Zuul]
---

# 问题描述

使用 Zuul 作为网关，偶发**超时问题**及**第一次调用触发熔断**问题

# 解决方案

## 超时问题

```
ribbon:
  ReadTimeout: 10000
  SocketTimeout: 60000
```

## 第一次调用触发熔断

```
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 10000
```

因为 Zuul 采用了懒加载机制，第一次访问的时候才会加载某些类，由于默认的时间原本就比较短，加载这些类又需要一些时间，造成超时


参考：

1. [Zuul超时问题，微服务响应超时，zuul进行熔断](https://blog.csdn.net/tianyaleixiaowu/article/details/78772269)
1. [官网: Zuul Timeouts](https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#_zuul_timeouts)
1. [Zuul超时配置](https://www.jianshu.com/p/8d7dc1c58346)
