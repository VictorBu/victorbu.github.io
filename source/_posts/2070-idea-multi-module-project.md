---
title: 使用 IDEA 创建多模块项目
date: 2019-05-20 17:00:00
updated: 2019-05-20 17:00:00
categories: [IT]
tags: [IDE, IDEA]
---

> 网上找如何创建多模块项目的资料，大多类似，实践中又各有问题，此文为摸索之后总结

最终项目结构如下：

![](https://oss.x8y.cc/blog-img/2070/0-0.png)

项目引用关系：app --> service --> dao

# 新建父项目 multi-parent

![](https://oss.x8y.cc/blog-img/2070/0-1.png)

![](https://oss.x8y.cc/blog-img/2070/0-2.png)

![](https://oss.x8y.cc/blog-img/2070/0-3.png)

multi-parent 项目只做模块的管理，不实现逻辑，所以可以删除 src 文件夹

# 新建模块项目 dao, service

在 multi-parent 右键

![](https://oss.x8y.cc/blog-img/2070/1-1.png)

其余步骤与新建父项目步骤一样

# 新建模块项目 app (Spring Boot 项目)

![](https://oss.x8y.cc/blog-img/2070/2-1.png)

![](https://oss.x8y.cc/blog-img/2070/2-2.png)

![](https://oss.x8y.cc/blog-img/2070/2-3.png)

![](https://oss.x8y.cc/blog-img/2070/2-4.png)

app 项目是 Spring Boot 项目，没有自动添加到 multi-parent 的 pom 中，需手动添加：

```
<modules>
    <module>dao</module>
    <module>service</module>
    <module>app</module>
</modules>
```

# 添加项目引用

## 在 service 的 pom 中添加

```
<dependency>
    <groupId>com.karonda</groupId>
    <artifactId>dao</artifactId>
    <version>1.0.0</version>
</dependency>
```

## 在 app 的 pom 中添加

```
<dependency>
    <groupId>com.karonda</groupId>
    <artifactId>service</artifactId>
    <version>1.0.0</version>
</dependency>
```

至此已完成多模块项目的创建，可以添加测试代码查看效果


测试代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/multi-parent)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

