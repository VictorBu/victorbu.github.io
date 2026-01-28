---
title: Java MQTT 客户端之 Paho
date: 2020-07-17 11:00:00
updated: 2020-07-17 11:00:00
categories: [IT]
tags: [Java, MQTT, Paho]
---

> Paho 自动重连后订阅的主题会清空，所以需要实现 MqttCallbackExtended 接口，在 connectComplete 方法添加订阅主题；而不是实现 MqttCallback 接口

# 一、添加引用

```
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.5</version>
</dependency>
```

# 二、添加配置

```
mqtt:
  client:
    username: admin
    password: public
    serverURI: tcp://192.168.137.101:1883
    clientId: paho_${random.int[1000,9999]}
    keepAliveInterval: 120
    connectionTimeout: 30
  producer:
    defaultQos: 1
    defaultRetained: true
    defaultTopic: topic/test1
  consumer:
    consumerTopics: topic/test2,topic/test3
```

# 三、代码

## 3.1.客户端

```
@Configuration
public class MqttConfig {
    @Value("${mqtt.client.username}")
    private String username;
    @Value("${mqtt.client.password}")
    private String password;
    @Value("${mqtt.client.serverURI}")
    private String serverURI;
    @Value("${mqtt.client.clientId}")
    private String clientId;
    @Value("${mqtt.client.keepAliveInterval}")
    private int keepAliveInterval;
    @Value("${mqtt.client.connectionTimeout}")
    private int connectionTimeout;

    @Autowired
    private MyMqttCallback myMqttCallback;

    @Bean
    public MqttClient mqttClient() {
        try {
            MqttClientPersistence persistence = mqttClientPersistence();
            MqttClient client = new MqttClient(serverURI, clientId, persistence);

            myMqttCallback.setMqttClient(client);
            client.setCallback(myMqttCallback);

            client.connect(mqttConnectOptions());
//            client.subscribe(subTopic);

            return client;
        } catch (MqttException e) {
            System.out.println(e.getMessage());
            return null;
        }
    }

    @Bean
    public MqttConnectOptions mqttConnectOptions() {
        MqttConnectOptions options = new MqttConnectOptions();
        options.setUserName(username);
        options.setPassword(password.toCharArray());
        options.setCleanSession(true);
        options.setAutomaticReconnect(true);
        options.setConnectionTimeout(connectionTimeout);
        options.setKeepAliveInterval(keepAliveInterval);

        return options;
    }

    public MqttClientPersistence mqttClientPersistence() {
        return new MemoryPersistence();
    }
}
```

## 3.2.订阅者

```
@Component
public class MyMqttCallback implements MqttCallbackExtended {

    @Value("${mqtt.consumer.consumerTopics}")
    private String[] consumerTopics;

    @Autowired
    private MqttService mqttService;

    private MqttClient mqttClient;

    @Override
    public void connectionLost(Throwable throwable) {
        System.out.println("连接断开");
    }

    @Override
    public void messageArrived(String topic, MqttMessage message) throws Exception {
        mqttService.message(topic, message);
    }

    @Override
    public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {
        System.out.println("deliveryComplete---------" + iMqttDeliveryToken.isComplete());
    }


    @Override
    public void connectComplete(boolean b, String s) {
        try {
            mqttClient.subscribe(consumerTopics);
        } catch (MqttException e) {
            System.out.println(e.getMessage());
        }
    }

    public void setMqttClient(MqttClient mqttClient) {
        this.mqttClient = mqttClient;
    }
}
```

## 3.3.发布者

```
@Component
public class MqttProducer {

    @Value("${mqtt.producer.defaultQos}")
    private int defaultProducerQos;
    @Value("${mqtt.producer.defaultRetained}")
    private boolean defaultRetained;
    @Value("${mqtt.producer.defaultTopic}")
    private String defaultTopic;

    @Autowired
    private MqttClient mqttClient;

    public void send(String payload) {
        this.send(defaultTopic, payload);
    }

    public void send(String topic, String payload) {
        this.send(topic, defaultProducerQos, payload);
    }

    public void send(String topic, int qos, String payload) {
        this.send(topic, qos, defaultRetained, payload);
    }

    public void send(String topic, int qos, boolean retained, String payload) {
        try {
            mqttClient.publish(topic, payload.getBytes(), qos, retained);
        } catch (MqttException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

```
@RestController
public class MqttController {

    @Autowired
    private MqttProducer mqttProducer;

    @RequestMapping("/send")
    public void send() {

        mqttProducer.send("test content");

    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/mqtt-paho)

> 参考

1. [MQTT Client in Java](https://www.baeldung.com/java-mqtt-client)
1. [MQTT Java 客户端库](https://docs.emqx.io/broker/latest/cn/development/java.html)
1. [使用paho的MQTT时遇到的重连导致订阅无法收到问题和解决](https://www.cnblogs.com/x-h-s/p/9455672.html)