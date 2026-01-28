---
title: Spring Cloud 学习 (六) Spring Cloud Config
date: 2019-06-14 12:30:00
updated: 2019-06-14 12:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Spring Cloud Config]
---

在实际开发过程中，每个服务都有大量的配置文件，例如数据库的配置、日志输出级别的配置等，而往往这些配置在不同的环境中也是不一样的。随着服务数量的增加，配置文件的管理也是一件非常复杂的事

在微服务架构中，需要有统一管理配置文件的组件，例如 Spring Cloud 的 Spring Cloud Config、阿里的 Diamond、百度的 Disconf、携程的 Apollo 等


新建 spring-cloud-config-server

# 从本地读取配置

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-config-server</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
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
  port: 8061

spring:
  application:
    name: config-server

  profiles:
    active: native # 从本地读取配置

  cloud:
    config:
      server:
        native:
          search-locations: classpath:/shared # 本地配置路径
```

## 新建配置文件 shared/eureka-client-dev.yml

```
version: 1.0.0
```

## 启动类

```
@EnableConfigServer // 开启 Config Server
@SpringBootApplication
public class ConfigServerApp {
    public static void main(String[] args){
        SpringApplication.run(ConfigServerApp.class, args);
    }
}
```

## 测试

启动 config-server

访问 http://localhost:8061/eureka-client/dev 可以看到配置

## 在客户端使用

在 eureka-client 测试

### 添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 添加配置 bootstrap.yml

```
spring:
  application:
    name: eureka-client

  cloud:
    config:
      uri: http://localhost:8061
      fail-fast: true

  profiles:
    active: dev
```

bootstrap.yml 优先于 application.yml

### 修改测试代码

```
@RestController
public class HiController {

    @Value("${server.port}")
    int port;

    @Value("${version}")
    String version;

    @GetMapping("/hi")
    public String home(@RequestParam String name){
        return "Hello " + name + ", from port: " + port + ", version: " + version;
    }
}
```

### 测试

1. 启动 config-server
1. 启动 eureka-server
1. 启动 eureka-client

访问 http://localhost:8011/hi?name=Victor


# 从远程 Git 仓库读取配置

## 修改 application.xml

```
server:
  port: 8061

spring:
  application:
    name: config-server

#  profiles:
#    active: native # 从本地读取配置

  cloud:
    config:
      server:
#        native:
#          search-locations: classpath:/shared # 本地配置路径
        git:
          uri: https://github.com/VictorBu/spring-cloud-config-demo
          search-paths: repo # 搜索的文件夹地址
#          username: # 如果是私有仓库须配置
#          password: # 如果是私有仓库须配置
      label: master # git 仓库分支名
```

## 测试

参考上节


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

