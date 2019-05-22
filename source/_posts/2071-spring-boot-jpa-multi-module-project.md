---
title: Spring Boot + JPA 多模块项目无法注入 JpaRepository 接口
date: 2019-05-22 11:00:00
updated: 2019-05-22 11:00:00
categories: [IT]
tags: [Spring Boot, JPA]
---

# 问题描述

Spring Boot + JPA 多模块项目，启动报异常：

```
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type '***.***.***Dao' available: 
expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: 
{@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

# 解决方案

在启动类添加 @ComponentScan, @EnableJpaRepositories, @EntityScan 注解：

```
@SpringBootApplication
@ComponentScan("***.***") // 1. 多模块项目需要扫描的包
@EnableJpaRepositories("***.***.***") // 2. Dao 层所在的包 
@EntityScan("***.***.***") // 3. Entity 所在的包
```

参考：[SpringBoot JPA 中无法注入 JpaRepository 接口@ComponentScan无效](https://blog.csdn.net/u012777670/article/details/83503200)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

