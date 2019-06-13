---
title: Spring Cloud 学习 (四) Hystrix & Hystrix Dashboard & Turbine
date: 2019-06-13 11:30:00
updated: 2019-06-13 11:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Hystrix, Ribbon, Feign, Hystrix Dashboard, Turbine]
---

在复杂的分布式系统中，可能有几十个服务相互依赖，这些服务由于某些原因，例如机房的不可靠性、网络服务商的不可靠性等，导致某个服务不可用 。 如果系统不隔离该不可用的服务，可能会导致整个系统不可用。Hystrix 提供了熔断器功能，能够阻止分布式系统中出现联动故障。Hystrix 是通过隔离服务的访问点阻止联动故障的，并提供了故障的解决方案，从而提高了整个分布式系统的弹性。

# 使用 Hystrix

## 在 Ribbon 使用 Hystrix

### 添加依赖包

```
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-javanica</artifactId>
</dependency>
```

### 修改启动类

```
@EnableHystrix // 启用 Hystrix 熔断器
@EnableEurekaClient
@SpringBootApplication
public class EurekaRibbonClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaRibbonClientApp.class, args);
    }
}
```

### 修改 Service

```
@Service
public class RibbonService {
    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "hiError") // 启用 Hystrix 熔断器
    public String hi(String name){
        return restTemplate.getForObject("http://eureka-client/hi?name=" + name, String.class);
    }

    public String hiError(String name){
        return "error! sorry, " + name;
    }
}
```

### 测试

1. 启动 eureka-server
1. 启动 eureka-ribbon-client

访问 http://localhost:8021/hi?name=victor 可以看到返回的是 hiError 方法执行的结果

## 在 Feign 使用 Hystrix

### application.yml 添加

```
feign:
  hystrix:
    enabled: true
```

### 添加熔断处理类

```
@Component
public class HiHystrix implements EurekaClientFeign {

    @Override
    public String sayHi(String name) {
        return "error! sorry, " + name;
    }
}
```

### 修改 FeignClient

```
@FeignClient(value = "eureka-client", configuration = FeignConfig.class
        , fallback = HiHystrix.class)
public interface EurekaClientFeign {

    @GetMapping("/hi")
    String sayHi(@RequestParam(value = "name") String name);
}
```

### 测试

1. 启动 eureka-server
1. 启动 eureka-feign-client

访问 http://localhost:8031/hi?name=victor 可以看到返回的是 hiError 方法执行的结果


# 使用 Hystrix Dashboard 监控熔断器状态

## 在 Ribbon 使用 Hystrix Dashboard

### 添加依赖包

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

### 修改启动类

```
@EnableHystrix // 启用 Hystrix 熔断器
@EnableHystrixDashboard // 启用 Hystrix Dashboard
@EnableEurekaClient
@SpringBootApplication
public class EurekaRibbonClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaRibbonClientApp.class, args);
    }
}
```

### application.yml 添加

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 测试

1. 启动 eureka-server
1. 启动 eureka-ribbon-client

先访问 http://localhost:8021/hi?name=victor，然后访问 http://localhost:8021/actuator/hystrix.stream 浏览器上会显示熔断器的数据指标

访问 http://localhost:8021/hystrix，然后在界面输入 http://localhost:8021/actuator/hystrix.stream 点击 [Monitor Stream] 按钮可以看到熔断器的各种数据指标

## 在 Feign 使用 Hystrix Dashboard

### 添加依赖包

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

与 Ribbon 相比需要添加 spring-cloud-starter-netflix-hystrix 依赖

### 修改启动类

```
@EnableFeignClients // 开启 Feign Client
@EnableHystrixDashboard // 启用 Hystrix Dashboard
@EnableCircuitBreaker // 启用 Hystrix 熔断器
@EnableEurekaClient
@SpringBootApplication
public class EurekaFeignClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaFeignClientApp.class, args);
    }
}
```

与 Ribbon 相比需要添加 @EnableCircuitBreaker 注解

### application.yml 添加

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 测试

1. 启动 eureka-server
1. 启动 eureka-feign-client

先访问 http://localhost:8031/hi?name=victor，然后访问 http://localhost:8031/actuator/hystrix.stream 浏览器上会显示熔断器的数据指标

访问 http://localhost:8031/hystrix，然后在界面输入 http://localhost:8031/actuator/hystrix.stream 点击 [Monitor Stream] 按钮可以看到熔断器的各种数据指标


# 使用 Turbine 聚合监控

新建 spring-cloud-eureka-turbine-client Module

## pom

```
<parent>
    <artifactId>spring-cloud-parent</artifactId>
    <groupId>com.karonda</groupId>
    <version>1.0.0</version>
</parent>
<modelVersion>4.0.0</modelVersion>

<artifactId>spring-cloud-eureka-turbine-client</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
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
  port: 8041


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8001/eureka/

spring:
  application:
    name: turbine-client
turbine:
  aggregator:
    cluster-config: default
  app-config: ribbon-client, feign-client
  cluster-name-expression: new String("default")

```

## 启动类

```
@SpringBootApplication
@EnableHystrixDashboard // 启用 Hystrix Dashboard
@EnableTurbine // 启用 Hystrix Turbine
public class EurekaTurbineClientApp {
    public static void main(String[] args){
        SpringApplication.run(EurekaTurbineClientApp.class, args);
    }
}
```

## 测试

1. 启动 eureka-server
1. 启动 eureka-ribbon-client
1. 启动 eureka-feign-client
1. 启动 eureka-turbine-client
1. 访问 http://localhost:8021/hi?name=victor
1. 访问 http://localhost:8031/hi?name=victor

访问 http://localhost:8021/hystrix 或 http://localhost:8031/hystrix 然后在界面输入 http://localhost:8041/turbine.stream 点击 [Monitor Stream] 按钮可以看到 eureka-ribbon-client 和 eureka-feign-client 熔断器的各种数据指标



完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

