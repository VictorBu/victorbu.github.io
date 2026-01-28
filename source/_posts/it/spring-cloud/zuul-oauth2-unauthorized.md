---
title: 解决 Zuul 中 OAuth2 报 unauthorized 错误
date: 2019-07-11 08:30:00
updated: 2019-07-11 08:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Zuul, OAuth2]
---

# 问题描述

微服务中使用 OAuth2 鉴权，直接访问正常，通过 Zuul 访问报错：

```
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

# 解决方法

在 Zuul 中添加配置：

```
zuul:
  sensitive-headers: Cookie,Set-Cookie
```

# 原因分析

zuul.sensitive-headers (默认值 Cookie,Set-Cookie,Authorization) 是指 http header 中的敏感信息，默认情况下，ZUUL 是不转发的

OAuth2 的鉴权信息是放在 Authorization 中，所以需要从配置中移除


参考：[Spring Cloud 随笔：记录在使用 OAuth2 遇到的巨坑](https://mp.weixin.qq.com/s/b1PZl8OVj3m7Q5IaJR_dpQ)

