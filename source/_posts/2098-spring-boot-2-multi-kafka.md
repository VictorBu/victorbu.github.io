---
title: Spring Boot 集成多个 Kafka
date: 2019-12-19 15:30:00
updated: 2019-12-19 15:30:00
categories: [IT]
tags: [Spring Boot, Kafka]
---

# 一、配置文件

application.yml

```
spring:
  kafka:
    one:
      bootstrap-servers: IP:PORT
      consumer:
        group-id: YOUR_GROUP_ID
        enable-auto-commit: true
    two:
      bootstrap-servers: IP:PORT
      consumer:
        group-id: YOUR_GROUP_ID
        enable-auto-commit: true
```

# 二、生产者、消费者配置

## 2.1 第一个 Kafka

```
@EnableKafka
@Configuration
public class KafkaOneConfig {

    @Value("${spring.kafka.one.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.one.consumer.group-id}")
    private String groupId;
    @Value("${spring.kafka.one.consumer.enable-auto-commit}")
    private boolean enableAutoCommit;


    @Bean
    public KafkaTemplate<String, String> kafkaOneTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>> kafkaOneContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }

    private ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    public ConsumerFactory<Integer, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    private Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        props.put(ProducerConfig.ACKS_CONFIG, "1"); // 不能写成 1
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    private Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }
}
```

## 2.2 第二个 Kafka

```
@Configuration
public class KafkaTwoConfig {

    @Value("${spring.kafka.two.bootstrap-servers}")
    private String bootstrapServers;
    @Value("${spring.kafka.two.consumer.group-id}")
    private String groupId;
    @Value("${spring.kafka.two.consumer.enable-auto-commit}")
    private boolean enableAutoCommit;

    @Bean
    public KafkaTemplate<String, String> kafkaTwoTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>> kafkaTwoContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }

    private ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    public ConsumerFactory<Integer, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    private Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        props.put(ProducerConfig.ACKS_CONFIG, "1");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    private Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, enableAutoCommit);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }
}
```

# 三、生产者

```
@Controller
public class TestController {

    @Autowired
    private KafkaTemplate kafkaOneTemplate;
    @Autowired
    private KafkaTemplate kafkaTwoTemplate;

    @RequestMapping("/send")
    @ResponseBody
    public String send() {
        final String TOPIC = "TOPIC_1";
        kafkaOneTemplate.send(TOPIC, "kafka one");
        kafkaTwoTemplate.send(TOPIC, "kafka two");

        return "success";
    }
}
```

# 四、消费者

```
@Component
public class KafkaConsumer {

    private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumer.class);

    final String TOPIC = "TOPIC_1";

    // containerFactory 的值要与配置中 KafkaListenerContainerFactory 的 Bean 名相同
    @KafkaListener(topics = {TOPIC}, containerFactory = "kafkaOneContainerFactory")
    public void listenerOne(ConsumerRecord<?, ?> record) {
        LOGGER.info(" kafka one 接收到消息：{}", record.value());
    }

    @KafkaListener(topics = {TOPIC}, containerFactory = "kafkaTwoContainerFactory")
    public void listenerTwo(ConsumerRecord<?, ?> record) {
        LOGGER.info(" kafka two 接收到消息：{}", record.value());
    }
}
```

# 运行结果

```
c.k.s.consumer.KafkaConsumer             :  kafka one 接收到消息：kafka one
c.k.s.consumer.KafkaConsumer             :  kafka two 接收到消息：kafka two
```


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-multi-kafka)

