---
title: 在 Spring Boot 配置 Kafka 安全认证
date: 2020-09-03 20:00:00
updated: 2020-09-03 20:00:00
categories: [IT]
tags: [Spring Boot, Kafka]
---

```
spring:
  kafka:
    bootstrap-servers: IP:端口
    listener:
      missing-topics-fatal: false
    properties:
      sasl:
        mechanism: PLAIN
        jaas:
          config: 'org.apache.kafka.common.security.plain.PlainLoginModule required username="用户名" password="密码";'
      security:
        protocol: SASL_PLAINTEXT
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: test_consumer_group
      enable-auto-commit: true
      auto-commit-interval: 1000
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

参考：[SpringBoot 支持Kafka安全认证 SASL/PLAINTEXT，账号密码认证](https://blog.csdn.net/u010637366/article/details/108142216)

