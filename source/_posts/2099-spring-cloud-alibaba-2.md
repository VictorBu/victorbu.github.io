---
title: Spring Cloud Alibaba 初体验(二) Nacos 服务注册与发现
date: 2020-03-27 10:00:00
updated: 2020-03-27 10:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Nacos]
---

# 一、服务注册

添加依赖：

```
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
	<version>2.2.0.RELEASE</version>
	<type>pom.sha256</type>
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

> 参考：

[Nacos Spring Cloud 快速开始](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)
