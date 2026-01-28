---
title: Spring Cloud Alibaba 初体验(五) SkyWalking
date: 2020-04-16 12:00:00
updated: 2020-04-16 12:00:00
categories: [IT]
tags: [Spring Cloud, Alibaba, SkyWalking]
---

# 一、下载与运行

本文使用 SkyWalking 7.0.0：https://www.apache.org/dyn/closer.cgi/skywalking/7.0.0/apache-skywalking-apm-7.0.0.tar.gz



Windows 环境下双击 bin/startup.bat 可以同时启动 收集器服务和 webapp，访问：http://localhost:8080

> SkyWalking webapp 默认端口是8080，可以修改为其他端口(webapp/webapp.yml)

# 二、使用

修改 agent/config/agent.config 中 agent.service_name 和 collector.backend_service

启动程序:

```
java -javaagent:D:\apache-skywalking\agent\skywalking-agent.jar -jar service1-0.0.1-SNAPSHOT.jar

```

也可以不修改 agent/config/agent.config 而是在启动时手动指定，如指定 agent.service_name：

```
java -javaagent:D:\apache-skywalking\agent\skywalking-agent.jar -Dskywalking.agent.service_name=Service1 -jar service1-0.0.1-SNAPSHOT.jar
```

> 注意

1. SkyWalking 默认使用 H2 数据库，正式使用需切换为 MySQL 或 Elasticsearch
1. 实测中发现看不到服务，但是在追踪中可以看到接口调用，还不知道是配置问题还是版本兼容问题或者其他问题