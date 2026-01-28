---
title: Spring Boot 集成 MQTT
date: 2019-12-03 17:30:00
updated: 2019-12-03 17:30:00
categories: [IT]
tags: [Java, MQTT, Spring Boot]
---

** 本文代码有些许问题，处理方案见：[解决 spring-integration-mqtt 频繁报 Lost connection 错误](https://www.cnblogs.com/victorbu/p/12986000.html)**

# 一、添加配置

```
spring:
  mqtt:
    client:
      username: 用户名
      password: 密码
      serverURIs: tcp://ip:port # 客户端地址，多个使用逗号隔开
      clientId: client0001 # ${random.value}
      keepAliveInterval: 30
      connectionTimeout: 30
    producer:
      defaultQos: 1
      defaultRetained: true
      defaultTopic: defaultTopicName
    consumer:
      defaultQos: 1
      completionTimeout: 30000
      consumerTopics: topic1,topic2 # 监听的 topic，多个使用逗号隔开
```

# 二、客户端配置

```
    /* 客户端 */
    @Bean
    public MqttConnectOptions getMqttConnectOptions() {
        MqttConnectOptions mqttConnectOptions = new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        mqttConnectOptions.setServerURIs(serverURIs);
        mqttConnectOptions.setKeepAliveInterval(keepAliveInterval);
        mqttConnectOptions.setConnectionTimeout(connectionTimeout);

        return mqttConnectOptions;
    }

    @Bean
    public MqttPahoClientFactory getMqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setConnectionOptions(getMqttConnectOptions());

        return factory;
    }
```

# 三、发布消息

## 3.1 配置

```
    @Bean
    public MessageChannel outboundChannel() {
        return new DirectChannel();
    }

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
```

## 3.2 消息推送接口类

```
@MessagingGateway(defaultRequestChannel = MqttConfig.OUTBOUND_CHANNEL)
public interface MqttSender {

    void sendToMqtt(String data);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, String payload);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) int qos, String payload);
}
```

## 3.3 测试

```
@RestController
public class TestController {

    @Autowired
    private MqttSender mqttSender;

    @RequestMapping("/send")
    private void send(String data){
        mqttSender.sendToMqtt(data);
    }
}
```

# 四、订阅消息

## 4.1 配置

```
    @Bean
    public MessageChannel inboundChannel() {
        return new DirectChannel();
    }

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

## 4.2 测试

```
@Component
public class MqttConsumer {
    private static final Logger LOGGER = LoggerFactory.getLogger(MqttConsumer.class);

    @Bean
    @ServiceActivator(inputChannel = MqttConfig.INBOUND_CHANNEL)
    public MessageHandler handler() {
        return message -> {
            String topic = message.getHeaders().get(MqttConfig.RECEIVED_TOPIC_KEY).toString();
            LOGGER.info("[{}]主题接收到消息:{}", topic, message.getPayload().toString());
        };
    }
}
```


> 注意事项

1. @ServiceActivator 和 @MessagingGateway 中绑定的 Channel 名，需与返回 MessageChannel 的 Bean 的方法名一样：

	如发布者绑定的 Channel 名为 outboundChannel，则需要有对应的方法，如下：

	```
	@Bean
	public MessageChannel outboundChannel() {
		return new DirectChannel();
	}
	```
1. 发布者与订阅者的 Channel 名不能相同
1. 连接服务器的超时时间和订阅的超时时间单位不一样

> 参考

1. [MQTT系列教程1（基本概念介绍）](https://www.hangge.com/blog/cache/detail_2347.html)
1. [SpringBoot - 集成MQTT教程1（发布消息）](https://www.hangge.com/blog/cache/detail_2610.html)
1. [SpringBoot - 集成MQTT教程2（订阅消息）](https://www.hangge.com/blog/cache/detail_2611.html)


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boog-mqtt)