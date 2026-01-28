---
title: Spring Cloud Alibaba 初体验(三) Nacos 与 Dubbo 集成
date: 2020-03-30 11:00:00
updated: 2020-04-16 23:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Nacos, Dubbo]
---

# 一、新建项目

新建项目，只放置接口，用于暴露 Dubbo 服务接口

```
public interface GreetingService {
    String greeting();
}
```

# 二、provider

本文以上文中的 Service1 作为 provider，以 Service2 作为 consumer


## 2.1 添加依赖

```
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>
```

## 2.2 实现接口

```
@Service
@Component
public class GreetingServiceImpl implements GreetingService {

    @Value("${server.port:0}")
    private Integer port;

    @Override
    public String greeting() {
        return "hello from port: " + port;
    }
}
```

@Service 为 org.apache.dubbo.config.annotation.Service (旧的版本为 com.alibaba.dubbo.config.annotation.Service)，此处的 @Service 只能供 Dubbo RPC 调用，如果需要像之前一样正常在 Controller 层调用需要再添加 @Component 注解 (或者直接不添加 @Component 在项目内也通过 Dubbo 调用)

## 2.3 新增配置

```
dubbo:
  scan:
    base-packages: com.karonda.service1.service.impl
  protocol:
    name: dubbo
    port: -1 # -1 表示端口自增
  registry:
    address: nacos://192.168.92.1:8848
```

# 三、consumer

## 3.1 调用服务


```
    @Reference
    private GreetingService greetingService;

    @RequestMapping("/dubboGreeting")
    public String dubboGreeting() {
        return greetingService.greeting();
    }
```

@Reference 为 org.apache.dubbo.config.annotation.Reference (旧的版本为 com.alibaba.dubbo.config.annotation.Reference)

## 3.2 新增配置

```
dubbo:
  protocol:
    name: dubbo
    port: -1
  registry:
    address: nacos://192.168.92.1:8848
  consumer:
    check: false # 不加此配置项目启动时可能会失败
```

启动服务，访问 http://localhost:8079/dubboGreeting 可以看到结果
