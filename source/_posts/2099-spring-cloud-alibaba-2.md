---
title: Spring Cloud Alibaba 初体验(二) Nacos 服务注册与发现 + 集成 Spring Cloud Gateway
date: 2020-03-27 10:00:00
updated: 2020-04-16 23:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Nacos, Spring Cloud Gateway]
---

# 一、服务注册

添加依赖：

```
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

新建 Spring Cloud 项目，在 bootstrap.yml 新增配置：

```
spring:
  application:
    name: service1

  cloud:
    nacos:
      discovery:
        server-addr: 192.168.92.1:8848
```

启动项目，可以在 Nacos 服务列表看到服务已注册

# 二、服务消费

新建项目 service2 (端口：8079)

在 bootstrap.yml 新增配置：

```
spring:
  application:
    name: service2

  cloud:
    nacos:
      discovery:
        server-addr: 192.168.92.1:8848
```

新增 Controller:

```
@RestController
public class GreetingController {

    @Autowired
    private RestTemplate restTemplate;

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @RequestMapping("/greeting")
    public String greeting() {
        return restTemplate.getForObject("http://service1/greeting", String.class);
    }
}
```

访问 http://localhost:8079/greeting 可以看到与访问 http://localhost:8070/greeting 同样的结果

# 三、集成 Spring Cloud Gateway

新建项目 (本文使用的 Spring Cloud 的版本为 Hoxton.SR3)：

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

配置：

```
server:
  port: 8081
  
spring:
  application:
    name: gateway
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.92.1:8848
        namespace: e5fc372c-ad66-4e0e-a353-a217d0a315ba
    gateway:
      routes:
        - id: service1
          uri: lb://service1 # lb 代表注册中心的服务
          predicates:
            - Path=/service1/** # 匹配的 URL
          filters:
            - StripPrefix=1 # URL 去掉的前缀个数
```

*--- 2020-05-26 更新开始 ---*

Spring Cloud Gateway 跨域设置

```
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedHeaders: '*'
            allowedOrigins: '*'
            allowedMethods: '*'
```

*--- 2020-05-26 更新结束 ---*

访问 http://localhost:8081/service1/greeting 即可路由到 service1 的 greeting 方法

> 参考：

[Nacos Spring Cloud 快速开始](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)
