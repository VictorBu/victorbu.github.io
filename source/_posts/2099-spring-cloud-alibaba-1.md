---
title: Spring Cloud Alibaba 初体验(一) Nacos 配置中心
date: 2020-03-26 10:00:00
updated: 2020-04-16 23:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, Nacos]
---

# 一、Nacos 下载与初始化配置

本文使用1.2.0，下载地址：[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

Nacos 单机模式支持持久化配置到 MySQL 数据库，修改 conf/application.properties 配置：

```
spring.datasource.platform=mysql

db.num=1

db.url.0=jdbc:mysql://数据库地址:端口/数据库名?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=用户名
db.password=密码
```

然后在数据库执行 conf/nacos-mysql.sql 初始化数据库

Windows 环境下双击 bin/startup.cmd 启动 Nacos

# 二、Nacos 配置及在 Spring Cloud 中的使用

Nacos 可以根据三种模式区分不同环境下的配置：

1. 根据 Data Id 和 profiles (spring.profiles.active)
1. 根据 Group
1. 根据 Namespace (命名空间)

本人认为根据 Namespace 区分不同环境下的配置应该更适用于实际项目，故本文以 Namespace 模式为例

## 2.1 新建命名空间

新增名字为 dev 的命名空间

## 2.2 新增配置

在 dev 命名空间新增配置，Data Id 为 service1.yml，Group 为 service1，配置格式为 YAML，内容为：

```
server:
  port: 8070
```

## 2.3 在 Spring Cloud 中使用

添加依赖：

```
    <dependencies>
		...
		
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
		
		...
    </dependencies>

    <dependencyManagement>
        <dependencies>
			...
			
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
			
			...
        </dependencies>
    </dependencyManagement>
```

spring-cloud-alibaba-dependencies 是 Spring Cloud Alibaba BOM，包含了 Spring Cloud Alibaba 的所有依赖的版本，使用 Nacos, Sentinel 等时不用再指定版本

新建 Spring Cloud 项目，在 bootstrap.yml 新增配置：

```
spring:
  application:
    name: service1

  cloud:
    nacos:
      config:
        server-addr: 192.168.92.1:8848 # Nacos 地址与端口
        namespace: e5fc372c-ad66-4e0e-a353-a217d0a315ba # 命名空间ID
        group: service1
        file-extension: yml
```

启动项目，可以看到项目使用的端口为在 Nacos 中配置的 8070 说明配置生效

## 2.4 扩展配置

有时我们想要将配置放置在不同的文件或者多个项目共用部分配置，则可以添加扩展配置

### 2.4.1 在 Nacos 添加配置

在 dev 命名空间新增配置，Data Id 为 greeting.yml，Group 为 common，配置格式为 YAML，内容为：

```
greeting: Hello, world!
```

在 dev 命名空间新增配置，Data Id 为 author.yml，Group 为 common，配置格式为 YAML，内容为：

```
author: karonda
```

### 2.4.2 Spring Cloud 配置

方法一：

```
spring:
  application:
    name: service1

  cloud:
    nacos:
      config:
        server-addr: 192.168.92.1:8848
        namespace: e5fc372c-ad66-4e0e-a353-a217d0a315ba
        group: service1
        file-extension: yml
        extension-configs[0]:
          data-id: greeting.yml
          group: common
          refresh: true # 配置修改后是否自动更新
        extension-configs[1]:
          data-id: author.yml
          group: common
          refresh: true
```

方法二：

```
spring:
  application:
    name: service1

  cloud:
    nacos:
      config:
        server-addr: 192.168.92.1:8848
        namespace: e5fc372c-ad66-4e0e-a353-a217d0a315ba
        group: service1
        file-extension: yml
        extension-configs:
          - data-id: greeting.yml
            group: common
            refresh: true
          - data-id: author.yml
            group: common
            refresh: true
```

### 2.4.3 测试


```
@RefreshScope # 配置自动更新
@RestController
public class GreetingController {

    @Value("${greeting:}")
    private String greetingStr;
    @Value("${author:}")
    private String authorStr;

    @RequestMapping("/greeting")
    public String greeting() {
        return greetingStr + " from " + authorStr;
    }
}
```

访问 http://localhost:8070/greeting

> 参考：

1. [Nacos支持三种部署模式](https://nacos.io/zh-cn/docs/deployment.html)
1. [Nacos学习笔记(五)---- NacosConfig配置](https://blog.csdn.net/Very666/article/details/97537530)
