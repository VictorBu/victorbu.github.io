---
title: 解决 JPA 插入 MySQL 时间与实际时间差 13 个小时问题
date: 2019-05-08 17:00:00
updated: 2019-05-08 17:00:00
categories: [IT]
tags: [JPA, MySQL]
---

# 问题描述

公司使用的阿里云数据库服务器，插入时间与实际时间差 13 个小时

执行 `show variables like "%time_zone%";` 结果如下：

| Variable_name | Value |
| ------ | ------ |
| system_time_zone | CST |
| time_zone | SYSTEM |

# 原因

CST 的时区是一个很混乱的时区，在与 MySQL 协商会话时区时，Java 会误以为是 CST -0500，而非 CST +0800，详见 [一次 JDBC 与 MySQL 因 “CST” 时区协商误解导致时间差了 14 或 13 小时的排错经历](https://juejin.im/post/5902e087da2f60005df05c3d)

# 解决方法

因此数据库为正式环境数据库，亦有其他程序正在使用，故在数据库连接字符串添加 `&serverTimezone=GMT%2b8`

另外一种方法需要修改 MySQL 配置：

```
[mysqld]
default-time-zone = '+08:00'
```


参考：

+ [一次 JDBC 与 MySQL 因 “CST” 时区协商误解导致时间差了 14 或 13 小时的排错经历](https://juejin.im/post/5902e087da2f60005df05c3d)
+ [spring data jpa + mysql设置时区为东八区](https://blog.csdn.net/CaptainJava/article/details/88723405)