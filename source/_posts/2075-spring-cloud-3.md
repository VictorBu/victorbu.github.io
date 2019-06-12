---
title: Spring Cloud 学习 (三) Feign
date: 2019-06-12 11:30:00
updated: 2019-06-12 11:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Feign]
---

新建 spring-cloud-eureka-feign-client Module

# pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-feign-client</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
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
  port: 8031


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: feign-client
```

# 启动类

```
@EnableFeignClients // 开启 Feign Client
@EnableEurekaClient
@SpringBootApplication
public class EurekaFeignClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaFeignClientApp.class, args);
    }
}
```

# FeignClient

```
@FeignClient("eureka-client")
public interface EurekaClientFeign {

    @GetMapping("/hi")
    String sayHi(@RequestParam(value = "name") String name);
}
```

# Controller

```
@RestController
public class FeignController {

    @Autowired
    EurekaClientFeign eurekaClientFeign;

    @GetMapping("/hi")
    public String sayHi(@RequestParam String name){
        return eurekaClientFeign.sayHi(name);
    }
}
```

# 测试

1. 启动 eureka-server
1. 启动 eureka-client (两个实例：一个 8011 端口，一个 8012 端口)
1. 启动 eureka-feign-client

多次访问 http://localhost:8031/hi?name=victor 可以看到 8011 和 8012 端口交替出现

# 添加失败重试

## 添加配置

```
@Configuration
public class FeignConfig {

    @Bean
    public Retryer feignRetryer(){
        return new Retryer.Default(100, SECONDS.toMillis(1), 5);
    }
}
```

## 修改 FeignClient

```
@FeignClient(value = "eureka-client", configuration = FeignConfig.class)
public interface EurekaClientFeign {

    @GetMapping("/hi")
    String sayHi(@RequestParam(value = "name") String name);
}
```

# 使用 HttpClient 或 OkHttp

在 Feign 中，Client 是一个非常重要的组件，Feign 最终发送 Request 请求以及接收 Response 响应都是由 Client 组件完成的。Client 在 Feign 源码中是一个接口，在默认的情况下， Client 的实现类是 Client.Default, Client.Default 是由 HttpURLConnnection 来实现网络请求的。Client 还支持 HttpClient 和 OkhHttp 来进行网络请求

直接添加 HttpClient 或 OkhHttp 依赖包，Feign 会自动使用对应的网络请求框架



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

