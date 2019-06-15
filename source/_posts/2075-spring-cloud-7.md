---
title: Spring Cloud 学习 (七) Spring Cloud Sleuth
date: 2019-06-15 10:30:00
updated: 2019-06-15 10:30:00
categories: [IT]
tags: [Microservices, Spring Cloud, Spring Cloud Sleuth, Zipkin]
---

微服务架构是一个分布式架构，微服务系统按业务划分服务单元，一个微服务系统往往有很多个服务单元。由于服务单元数量众多，业务的复杂性较高，如果出现了错误和异常，很难去定位。主要体现在一个请求可能需要调用很多个服务，而内部服务的调用复杂性决定了问题难以定位。所以在微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题能够快速定位的目的

常见的链路追踪组件有 Google 的 Dapper、Twitter 的 Zipkin，以及阿里的 Eagleeye (鹰眼)

# 概念

1. Span: 基本工作单元，发送一个远程调度任务就会产生一个 Span。包含了摘要、时间戳事件、 Span 的 ID 以及进程 ID
1. Trace: 由一系列 Span 组成的，呈树状结构。请求一个微服务系统的 API 接口，这个 API 接口需要调用多个微服务单元，调用每个微服务单元都会产生一个新的 Span，所有由这个请求产生的 Span 组成了这个 Trace
1. Annotation: 用于记录一个事件，一些核心注解用于定义一个请求的开始和结束，这些注解如下：
    1. cs-Client Sent: 客户端发送一个请求，这个注解描述了 Span 的开始
    1. sr-Server Received: 服务端获得请求并准备开始处理它，用 sr 减去 cs 时间戳，便可得到网络传输的时间
    1. ss-Server Sent: 服务端发送响应，该注解表明请求处理的完成 (当请求返回客户端)，用 ss 的时间戳减去 sr 时间戳，便可以得到服务器请求的时间
    1. er-Client Received: 客户端接收响应，此时 Span 结束，用 er 的时间戳减去 cs 时间戳，便可以得到整个请求所消耗的时间

# Zipkin Server

下载 [Zipkin Server](https://search.maven.org/remote_content?g=io.zipkin.java&a=zipkin-server&v=LATEST&c=exec)

启动 Zipkin Server：

```
java -jar zipkin-server-2.12.9-exec.jar
```

Zipkin Server 默认端口为 9411

更详细关于 Zipkin 参考：https://zipkin.io/pages/quickstart.html

# eureka-client

## 添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

## application.yml 添加

```
spring:

  zipkin:
    base-url: http://localhost:9411 # Zipkin Server 地址
  sleuth:
    sampler:
      probability: 1.0 # 以 100% 的概率将链路的数据上传给 Zipkin Server
```

# zuul-client

步骤同 eureka-client

# 测试

1. 启动 eureka-server
1. 启动 config-server
1. 启动 eureka-client
1. 启动 zuul-client

访问 http://localhost:8051/hiapi/hi?name=victor&token=xx

访问 http://localhost:9411/zipkin 点击 [查找] 按钮即可看到链路信息


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-cloud-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

