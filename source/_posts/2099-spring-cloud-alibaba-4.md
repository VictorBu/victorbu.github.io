---
title: Spring Cloud Alibaba 初体验(四) Sentinel
date: 2020-03-30 17:00:00
updated: 2020-04-16 23:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Sentinel]
---

# 一、Sentinel 下载与运行

本文使用 Sentinel 1.7.1：https://github.com/alibaba/Sentinel/releases

使用自定义端口 8089 运行 Sentinel：

```
java -Dserver.port=8089 -Dcsp.sentinel.dashboard.server=localhost:8089 -jar sentinel-dashboard-1.7.1.jar
```

# 二、使用

## 2.1 添加引用

```
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

## 2.2 标识资源

使用 @SentinelResource 注解用来标识资源是否限流、降级。@SentinelResource 还提供了其它额外的属性如 blockHandler，blockHandlerClass，fallback 用于表示限流或降级的操作，更多内容可以参考 Sentinel [注解支持文档](https://github.com/alibaba/Sentinel/wiki/注解支持)

```
	@SentinelResource(value = "greeting")
    @Override
    public String greeting() {
        return "hello from port: " + port;
    }
```

## 2.3 添加配置

```
spring:
  application:
    name: service1

  cloud:
	... ...

    sentinel:
      transport:
        dashboard: localhost:8089
```

## 2.4 查看与操作

访问 http://localhost:8089 用户名、密码均为 sentinel，可以看到 Service1 的实时监控数据，可以添加限流(流控)的策略