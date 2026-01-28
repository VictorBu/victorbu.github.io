---
title: Spring Cloud 学习 (一) Eureka
date: 2019-06-11 11:20:00
updated: 2019-06-12 11:20:00
categories: [IT]
tags: [Microservices, Spring Cloud, Eureka]
---

微服务的功能主要有以下几个方面：

+ 服务的注册和发现
+ 服务的负载均衡
+ 服务的容错
+ 服务网关
+ 服务配置的统一管理
+ 链路追踪
+ 实时日志

**服务注册**是指向服务注册中心注册一个服务实例，服务提供者将自己的服务信息 (如服务名、IP 地址等) 告知服务注册中心。**服务发现**是指当服务消费者需要消费另外一个服务时，服务注册中心能够告知服务消费者它所要消费服务的实例信息(如服务名、IP 地址等)。通常情况下一个服务既是服务提供者，也是服务消费者。服务消费者一般使用 HTTP 协议或者消息组件这种轻量级的通信机制来进行服务消费。服务注册中心会提供服务的健康检查方案，检查被注册的服务是否可用。通常一个服务实例注册后，会定时向服务注册中心提供 “心跳”，以表明自己还处于可用的状态。当一个服务实例停止向服务注册中心提供心跳一段时间后，服务注册中心会认为该服务实例不可用，会将该服务实例从服务注册列表中剔 除。如果这个被剔除掉的服务实例过一段时间后继续向注册中心提供心跳，那么服务注册中心会将该服务实例重新加入服务注册中心的列表中。

Eureka 是一个用于服务注册和发现的组件，分为 Server (服务注册中心) 和 Client (服务提供者、服务消费者)。与 Eureka 类似得组件有 Consul 和 ZooKeeper。

Eureka 是 Spring Cloud 首选推荐的服务注册与发现组件，可以与 Spring Cloud 其他组件可以无缝对接。

# Eureka Server

新建一个多模块项目 (可参考：[使用 IDEA 创建多模块项目](/2019/05/2070-idea-multi-module-project/))，添加 spring-cloud-eureka-server 模块

其中父模块 pom：

```
<modelVersion>4.0.0</modelVersion>
<packaging>pom</packaging>
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<groupId>com.karonda</groupId>
<artifactId>spring-cloud-parent</artifactId>
<version>1.0.0</version>
<name>spring-cloud-parent</name>
<description>spring cloud parent project</description>

<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## pom：

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-server</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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

## application.yml

```
server:
  port: 8001

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false # 默认情况下 Eureka Server 会向自己注册，设为 false 防止自己注册自己
    fetch-registry: false # 默认情况下 Eureka Server 会向自己注册，设为 false 防止自己注册自己
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: eureka-server
```

## 启动类

```
@EnableEurekaServer // 开启 Eureka Server
@SpringBootApplication
public class EurekaServerApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaServerApp.class, args);
    }
}
```

# Eureka Client

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-client</artifactId>


<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
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

## application.yml

```
server:
  port: 8011


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: eureka-client
```

## 启动类

```
@EnableEurekaClient // 开启 Eureka Client
@SpringBootApplication
public class EurekaClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaClientApp.class, args);
    }
}
```

## 测试代码

```
@RestController
public class HiController {

    @Value("${server.port}")
    int port;

    @GetMapping("/hi")
    public String home(@RequestParam String name){
        return "Hello " + name + ", from port: " + port;
    }
}
```

# 测试

分别启动 Server 和 Client，在浏览器输入 http://localhost:8001/ 可以看到 Client 已经注册到 Server

中英对照：

+ 服务注册中心：Register Service
+ 服务提供者：Provider Service
+ 服务消费者：Consumer Service



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

