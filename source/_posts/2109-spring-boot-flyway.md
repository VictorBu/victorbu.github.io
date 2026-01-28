---
title: 在 Spring Boot 中使用 Flyway
date: 2020-05-26 15:00:00
updated: 2020-05-26 15:00:00
categories: [IT]
tags: [Spring Boot, Flyway]
---

# 一、Flyway 介绍

Flyway 是一个开源的数据库迁移工具，MySQL, SQL Server, Oracle 等二十多种数据库

在 Flyway 中数据库的所有改变均称为迁移(migration)，迁移分为两种：基于版本控制的迁移(versioned)和可重复执行的迁移(repeatable)。基于版本控制的又分为两种：常规迁移(regular)和撤销迁移(undo)

**基于版本控制的迁移**

包含版本(version)、描述(description )和总和校验码(checksum)。版本必须是唯一的(可以是递增的数字或者小版本，如1.2.3等)，总和校验码的作用是在 Flyway 会逐个对比之前执行过的脚本的是否被修改，如果被修改则校验码结果不同，会报 checksum mismatch 错误，因此执行过的脚本不应该再修改，如果需要对已有数据表修改，应该新建一个脚本执行；基于版本控制的迁移仅按照版本的顺序执行一次，如果需要撤销，可以提供相同版本的撤销迁移来处理

**可重复执行的迁移**

包含描述(description )和总和校验码(checksum)，但是没有版本(version)。每当校验码发生变化(即文件被修改)时均会执行，而不是仅执行一次；在单次迁移中，可重复执行的迁移按照描述的顺序在其他脚本执行后再执行


迁移的实现方式又分为两种：基于 SQL 脚本(SQL-based)的迁移和基于 Java 代码(Java-based)的迁移

**基于 SQL 脚本的迁移**

命名规则：[前缀][版本号][分隔符][描述][后缀]，如：V2__Add_new_table.sql，R__Add_new_table.sql

+ 前缀：常规迁移是 V，撤销迁移是 U，可重复执行的迁移 是 R;前缀是可配置的
+ 版本号：使用下划线或者小数点分割
+ 分隔符：默认是两个下划线(可配置)
+ 描述：使用下划线或者空格分割
+ 后缀：默认是 .sql (可配置)

配置参数 validateMigrationNaming 用来配置命名不符合规范的处理方式：true 忽略不符合规范的文件，false 快速失败

**基于 Java 代码的迁移**

命名规则：[前缀][版本号][分隔符][描述]，如：V2__Add_new_table，R__Add_new_table

+ 前缀：常规迁移是 V，撤销迁移是 U，可重复执行的迁移 是 R;前缀是可配置的
+ 版本号：使用下划线分割(下划线在运行时会自动替换为小数点)
+ 分隔符：两个下划线
+ 描述：使用下划线分割(下划线在运行时会自动替换为空格)


以上大部分内容为本人翻译自官网，原文：[Migrations](https://flywaydb.org/documentation/migrations)

# 二、在 Spring Boot 中使用

本文使用基于 SQL 脚本的迁移

## 2.1 添加依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.19</version>
</dependency>

<dependency>
	<groupId>org.flywaydb</groupId>
	<artifactId>flyway-core</artifactId>
</dependency>
```

## 2.2 添加配置

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/flyway?characterEncoding=utf-8&allowMultiQueries=true&serverTimezone=Asia/Shanghai
    username: root
    password: root

  flyway:
    enabled: true
    clean-disabled: true # 生产环境一定要设置为 true
    clean-on-validation-error: false
  #    locations: classpath:db/migration # 脚本位置，默认是 classpath:db/migration
```

在 db/migration 目录下添加脚本运行程序即会自动执行脚本