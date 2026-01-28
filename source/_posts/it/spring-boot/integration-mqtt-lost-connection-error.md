---
title: 解决 spring-integration-mqtt 频繁报 Lost connection 错误
date: 2020-05-29 12:00:00
updated: 2020-05-29 12:00:00
categories: [IT]
tags: [Spring Boot, MQTT]
---

# 问题描述

在之前的博客介绍了如何在 [Spring Boot 集成 MQTT](https://www.cnblogs.com/victorbu/p/11978107.html)，后面使用中没有发现问题，最近发现一直报错：

```
Lost connection: Connection lost; retrying...
```

```
Lost connection: 已断开连接; retrying...
```

# 解决过程

网上说是因为 client ID 重复，最开始是不相信的，因为我测试只启动了一个客户端。但是却怎么都定位不到异常原因，用重新回到  client ID 重复的这个思路上来：

因为程序里同时作为订阅者和发布者，就怀疑订阅和发布服务是不是单独建立的连接，抱着试试看的想法试了一下，结果果然是这个原因，原代码：

```
    /* 发布者 */
    @Bean
    @ServiceActivator(inputChannel = OUTBOUND_CHANNEL)
    public MessageHandler getMqttProducer() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(clientId, getMqttClientFactory());
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(defaultTopic);
        messageHandler.setDefaultRetained(defaultRetained);
        messageHandler.setDefaultQos(defaultProducerQos);

        return messageHandler;
    }

    /* 订阅者 */
    @Bean
    public MessageProducer getMqttConsumer() {
        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter(clientId, getMqttClientFactory(), consumerTopics);
        adapter.setCompletionTimeout(completionTimeout);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(defaultConsumerQos);
        adapter.setOutputChannel(inboundChannel());

        return adapter;
    }
```

订阅者和发布者使用的是相同的 client ID，修改后代码：

```
    /* 发布者 */
    @Bean
    @ServiceActivator(inputChannel = OUTBOUND_CHANNEL)
    public MessageHandler getMqttProducer() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(clientId + "_producer", getMqttClientFactory());
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(defaultTopic);
        messageHandler.setDefaultRetained(defaultRetained);
        messageHandler.setDefaultQos(defaultProducerQos);

        return messageHandler;
    }

    /* 订阅者 */
    @Bean
    public MessageProducer getMqttConsumer() {
        MqttPahoMessageDrivenChannelAdapter adapter =
                new MqttPahoMessageDrivenChannelAdapter(clientId + "_consumer", getMqttClientFactory(), consumerTopics);
        adapter.setCompletionTimeout(completionTimeout);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(defaultConsumerQos);
        adapter.setOutputChannel(inboundChannel());

        return adapter;
    }
```

# 总结

虽然目前解决了这个问题，但是为什么会单独建立两个连接的原因还未找到；另外，一个程序两个连接还是感觉怪怪的，不知道还有没有更优的处理方案
