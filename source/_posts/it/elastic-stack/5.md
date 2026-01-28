---
title: 浅尝 Elastic Stack (五) Logstash + Beats + Kafka
date: 2020-06-05 11:00:00
updated: 2020-06-05 11:00:00
categories: [IT]
tags: [Elastic, Logstash, Beats, Kafka]
---

在 [Elasticsearch、Kibana、Beats 安装](https://www.cnblogs.com/victorbu/p/13026455.html) 中讲到推荐架构：

![](https://oss.x8y.cc/blog-img/2111/v2-3b578fddb1221bcdb250fcfdf3b070a2_r.jpg)

本文基于 [Logstash + Beats 读取 Spring Boot 日志](https://www.cnblogs.com/victorbu/p/13046532.html) 将其改为上述架构

如果没有安装 Kafka 需要首先安装：http://kafka.apache.org/quickstart ，如果需要后台运行添加 -daemon 即可

# 一、FileBeat

修改 output

```
output.kafka:
  hosts: ["localhost:9092"]
  topic: demo
```

# 二、Logstash

修改 input

```
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => "demo"
  }
}
```