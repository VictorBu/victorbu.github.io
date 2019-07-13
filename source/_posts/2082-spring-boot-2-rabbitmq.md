---
title: Spring Boot + RabbitMQ 使用示例
date: 2019-07-13 18:30:00
updated: 2019-07-13 18:30:00
categories: [IT]
tags: [Spring Boot, RabbitMQ]
---

# 基础知识

1. 虚拟主机 (Virtual Host): 每个 virtual host 拥有自己的 exchanges, queues 等 (类似 MySQL 中的库)
1. 交换器 (Exchange): 生产者产生的消息并不是直接发送给 queue 的，而是要经过 exchange 路由, exchange 类型如下：
    1. fanout: 把所有发送到该 exchange 的消息路由到所有与它绑定的 queue 中
    1. direct: 把消息路由到 binding key 与routing key 完全匹配的 queue 中
    1. topic: 模糊匹配 (单词间使用"."分割，"*" 匹配一个单词，"#" 匹配零个或多个单词)
    1. headers: 根据发送的消息内容中的 headers 属性进行匹配
1. 信道 (Channel): 建立在真实的 TCP 连接之上的虚拟连接, RabbitMQ 处理的每条 AMQP 指令都是通过 channel 完成的

# 使用示例

RabbitMQ 安装参考： [docker 安装rabbitMQ](https://www.cnblogs.com/yufeng218/p/9452621.html)

新建 Spring Boot 项目，添加配置：

```
spring:
  rabbitmq:
    host: 192.168.30.101
    port: 5672
    username: admin
    password: admin
    virtual-host: my_vhost

logging:
  level:
    com: INFO
```

## 1. [基本使用](https://www.rabbitmq.com/tutorials/tutorial-one-spring-amqp.html)

Queue

```
@Configuration
public class RabbitmqConfig {

    @Bean
    public Queue hello() {
        return new Queue("hello");
    }

}
```

Producer

```
@Component
@EnableAsync
public class SenderTask {

    private static final Logger logger = LoggerFactory.getLogger(SenderTask.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private Queue queue;

    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void send(){
        String message = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

        rabbitTemplate.convertAndSend(queue.getName(), message);

        logger.info(" [x] Sent '" + message + "'");
    }
}
```

Consumer

```
@Component
@RabbitListener(queues = "hello")
public class ReceiverTask {

    private static final Logger logger = LoggerFactory.getLogger(ReceiverTask.class);

    @RabbitHandler
    public void receive(String in){
        logger.info(" [x] Received '" + in + "'");
    }
}
```

## 2. [fanout](https://www.rabbitmq.com/tutorials/tutorial-three-spring-amqp.html)

Exchange, Queue, Binding

```
@Configuration
public class RabbitmqConfig {

    @Bean
    public FanoutExchange fanout() {
        return new FanoutExchange("fanoutExchangeTest");
    }

    @Bean
    public Queue autoDeleteQueue1() {
        return new AnonymousQueue();// 创建一个非持久的，独占的自动删除队列
    }

    @Bean
    public Queue autoDeleteQueue2() {
        return new AnonymousQueue();
    }

    @Bean
    public Binding binding1(FanoutExchange fanout,
                            Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1).to(fanout);
    }

    @Bean
    public Binding binding2(FanoutExchange fanout,
                            Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2).to(fanout);
    }
}
```

Producer

```
@Component
@EnableAsync
public class SenderTask {

    private static final Logger logger = LoggerFactory.getLogger(SenderTask.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private FanoutExchange fanoutExchange;

    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void send(){
        String message = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

        rabbitTemplate.convertAndSend(fanoutExchange.getName(), "", message);

        logger.info(" [x] Sent '" + message + "'");
    }
}
```

Consumer

```
@Component
public class ReceiverTask {

    private static final Logger logger = LoggerFactory.getLogger(ReceiverTask.class);

    @RabbitListener(queues = "#{autoDeleteQueue1.name}")
    public void receive1(String in){
        receive(in, 1);
    }

    @RabbitListener(queues = "#{autoDeleteQueue2.name}")
    public void receive2(String in){
        receive(in, 2);
    }

    public void receive(String in, int receiver){
        logger.info("instance " + receiver + " [x] Received '" + in + "'");
    }
}
```

## 3. [direct](https://www.rabbitmq.com/tutorials/tutorial-four-spring-amqp.html)

Exchange, Queue, Binding

```
@Configuration
public class RabbitmqConfig {

    @Bean
    public DirectExchange direct() {
        return new DirectExchange("directExchangeTest");
    }

    @Bean
    public Queue autoDeleteQueue1() {
        return new AnonymousQueue();// 创建一个非持久的，独占的自动删除队列
    }

    @Bean
    public Queue autoDeleteQueue2() {
        return new AnonymousQueue();
    }

    @Bean
    public Binding binding1a(DirectExchange direct,
                            Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1).to(direct).with("orange");
    }

    @Bean
    public Binding binding1b(DirectExchange direct,
                             Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1).to(direct).with("green");
    }

    @Bean
    public Binding binding2a(DirectExchange direct,
                             Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2).to(direct).with("green");
    }

    @Bean
    public Binding binding2b(DirectExchange direct,
                             Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2).to(direct).with("black");
    }
}
```

Producer

```
@Component
@EnableAsync
public class SenderTask {

    private static final Logger logger = LoggerFactory.getLogger(SenderTask.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private DirectExchange directExchange;

    private final String[] keys = {"orange", "black", "green"};

    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void send(){

        Random random = new Random();

        String key = keys[random.nextInt(keys.length)];

        String message = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))
                + " to: " + key;

        rabbitTemplate.convertAndSend(directExchange.getName(), key, message);

        logger.info(" [x] Sent '" + message + "'");
    }
}
```

Consumer

```
@Component
public class ReceiverTask {

    private static final Logger logger = LoggerFactory.getLogger(ReceiverTask.class);

    @RabbitListener(queues = "#{autoDeleteQueue1.name}")
    public void receive1(String in){
        receive(in, 1);
    }

    @RabbitListener(queues = "#{autoDeleteQueue2.name}")
    public void receive2(String in){
        receive(in, 2);
    }

    public void receive(String in, int receiver){
        logger.info("instance " + receiver + " [x] Received '" + in + "'");
    }
}
```

## 4. [topic](https://www.rabbitmq.com/tutorials/tutorial-five-spring-amqp.html)

Exchange, Queue, Binding

```
@Configuration
public class RabbitmqConfig {

    @Bean
    public TopicExchange topic() {
        return new TopicExchange("topicExchangeTest");
    }

    @Bean
    public Queue autoDeleteQueue1() {
        return new AnonymousQueue();// 创建一个非持久的，独占的自动删除队列
    }

    @Bean
    public Queue autoDeleteQueue2() {
        return new AnonymousQueue();
    }

    @Bean
    public Binding binding1a(TopicExchange topic,
                            Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1).to(topic).with("*.orange.*");
    }

    @Bean
    public Binding binding1b(TopicExchange topic,
                             Queue autoDeleteQueue1) {
        return BindingBuilder.bind(autoDeleteQueue1).to(topic).with("*.*.rabbit");
    }

    @Bean
    public Binding binding2a(TopicExchange topic,
                             Queue autoDeleteQueue2) {
        return BindingBuilder.bind(autoDeleteQueue2).to(topic).with("lazy.#");
    }

}
```

Producer

```
@Component
@EnableAsync
public class SenderTask {

    private static final Logger logger = LoggerFactory.getLogger(SenderTask.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private TopicExchange topicExchange;

    private final String[] keys = {"quick.orange.rabbit", "lazy.orange.elephant", "quick.orange.fox",
            "lazy.brown.fox", "lazy.pink.rabbit", "quick.brown.fox"};

    @Async
    @Scheduled(cron = "0/1 * * * * ? ")
    public void send(){

        Random random = new Random();

        String key = keys[random.nextInt(keys.length)];

        String message = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"))
                + " to: " + key;

        rabbitTemplate.convertAndSend(topicExchange.getName(), key, message);

        logger.info(" [x] Sent '" + message + "'");
    }
}
```

Consumer

```
@Component
public class ReceiverTask {

    private static final Logger logger = LoggerFactory.getLogger(ReceiverTask.class);

    @RabbitListener(queues = "#{autoDeleteQueue1.name}")
    public void receive1(String in){
        receive(in, 1);
    }

    @RabbitListener(queues = "#{autoDeleteQueue2.name}")
    public void receive2(String in){
        receive(in, 2);
    }

    public void receive(String in, int receiver){
        logger.info("instance " + receiver + " [x] Received '" + in + "'");
    }
}
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-rabbitmq)

