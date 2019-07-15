---
title: Feign 超时设置
date: 2019-07-15 18:30:00
updated: 2019-07-15 18:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Feign]
---

# 问题描述

微服务之间使用 Feign 调用，偶发超时问题，配置如下：

```
feign:
  client:
    config:
      default:
        connectTimeout: 10000
        readTimeout: 10000
```

详细参考官方文档：https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-feign.html

