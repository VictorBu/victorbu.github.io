---
title: Spring Cloud 学习 (二) Ribbon
date: 2019-06-12 11:20:00
updated: 2019-06-12 11:20:00
categories: [IT]
tags: [Microservices, Spring Cloud, Ribbon]
---

负载均衡是指将负载分摊到多个执行单元上，常见的负载均衡有两种方式：一种是独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，例如 Ngnix；另一种是将负载均衡逻辑以代码的形式封装到服务消费者的客户端上，服务消费者客户端维护了一份服务提供者的信息列表，有了信息列表，通过负载均衡策略将请求分摊给多个服务提供者，从而达到负载均衡的目的，例如 Ribbon


新建 spring-cloud-eureka-ribbon-client Module

# pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-ribbon-client</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

# application.yml

```
server:
  port: 8021


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: ribbon-client
```

# 启动类

```
@EnableEurekaClient
@SpringBootApplication
public class EurekaRibbonClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaRibbonClientApp.class, args);
    }
}
```

# RestTemplate

```
@Configuration
public class RibbonConfig {
    @Bean
    @LoadBalanced // 开启负载均衡功能
    RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

# Service

```
@Service
public class RibbonService {
    @Autowired
    RestTemplate restTemplate;

    public String hi(String name){
        return restTemplate.getForObject("http://eureka-client/hi?name=" + name, String.class);
    }
}
```

# Controller

```
@RestController
public class RibbonController {
    @Autowired
    RibbonService ribbonService;

    @GetMapping("/hi")
    public String hi(@RequestParam String name){
        return ribbonService.hi(name);
    }
}
```

# 测试

1. 启动 eureka-server
1. 启动 eureka-client (两个实例：一个 8011 端口，一个 8012 端口)
1. 启动 eureka-ribbon-client

多次访问 http://localhost:8021/hi?name=victor 可以看到 8011 和 8012 端口交替出现


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

