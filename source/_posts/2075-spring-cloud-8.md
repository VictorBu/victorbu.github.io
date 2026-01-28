---
title: Spring Cloud 学习 (八) Spring Boot Admin
date: 2019-06-16 10:30:00
updated: 2019-06-16 10:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Spring Boot Admin]
---

Spring Boot Admin 用于管理和监控一个或者多个 Spring Boot 程序

# 新建 spring-boot-admin-server

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-boot-admin-server</artifactId>

<dependencies>
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

spring-boot-admin-starter-server 需要指定版本号

## application.yml

```
server:
  port: 8071


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: admin-server
```

## 启动类

```
@EnableAdminServer // 开启 Admin Server
@EnableEurekaClient
@SpringBootApplication
public class AdminServerApp {
    public static void main(String[] args){
        SpringApplication.run(AdminServerApp.class, args);
    }
}
```

# eureka-client

## 添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

注册 Spring Boot Admin (SBA) 客户端有两种方式：一种是通过引入 SBA Client (spring-boot-admin-starter-client)；另一种是基于 Spring Cloud Discovery, 不需要添加依赖

## application.xml 添加

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: ALWAYS
```

# 测试

1. 启动 eureka-server
1. 启动 config-server
1. 启动 eureka-client
1. 启动 admin-server

访问 http://localhost:8071



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

