---
title: Spring Boot + Redis 初体验
date: 2019-05-15 17:00:00
updated: 2019-05-15 17:00:00
categories: [IT]
tags: [Java, Spring Boot, Redis]
---

> 本文测试环境: Spring Boot 2.1.4.RELEASE + Redis 5.0.4 + CentOS 7

# 让程序先 run 起来

## 安装及配置 Redis

参考: [How To Install and Configure Redis on CentOS 7](https://linuxize.com/post/how-to-install-and-configure-redis-on-centos-7/)

## 新建 Spring Boot 项目并添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 添加 Redis 配置

```
spring:
  redis:
    host: 192.168.30.101
    port: 6379
    database: 0
#    password: ******
    jedis:
      pool:
        max-active: 10
        min-idle: 0
        max-idle: 6
        max-wait: -1
```

## 测试代码

```
@Component
public class StringRunner implements CommandLineRunner {

    @Autowired
    private StringRedisTemplate redisTemplate;
//    private RedisTemplate<String, String> redisTemplate;

    @Override
    public void run(String... args) throws Exception {

        String key;

        // String: 字符串
        System.out.println("====== String ======");
        key = "string:string";
        redisTemplate.opsForValue().set(key, "string.string", 10, TimeUnit.SECONDS);
        redisTemplate.opsForValue().append(key, ".append");
        System.out.println(redisTemplate.opsForValue().get(key));

        key = "string:number";
        redisTemplate.delete(key);
        redisTemplate.opsForValue().increment(key);
        redisTemplate.opsForValue().decrement(key);
        System.out.println(redisTemplate.opsForValue().get(key));

        // Hash: 哈希
        System.out.println("====== Hash ======");
        key = "hash:test";
        redisTemplate.delete(key);
        redisTemplate.opsForHash().put(key, "key1", "value1");

        Map<String, String> map = new HashMap<>();
        map.put("key2", "value2");
        map.put("key3", "1");
        redisTemplate.opsForHash().putAll(key, map);

        System.out.println(redisTemplate.opsForHash().entries(key));

        System.out.println(redisTemplate.opsForHash().keys(key));

        System.out.println(redisTemplate.opsForHash().values(key));

        System.out.println(redisTemplate.opsForHash().get(key, "key1"));

        System.out.println(redisTemplate.opsForHash().increment(key, "key3", 1L));

        redisTemplate.opsForHash().delete(key, "key2");

        // List: 列表
        System.out.println("====== List ======");
        key = "list:test";
        redisTemplate.delete(key);
        redisTemplate.opsForList().rightPush(key, "1");
        redisTemplate.opsForList().leftPush(key, "2");
        redisTemplate.opsForList().rightPushAll(key, new String[]{"3", "4", "5"});
        List<String> list = redisTemplate.opsForList().range(key, 0, -1);
        for(String item : list){
            System.out.print(item + ", ");
        }
        System.out.println();

        // Set: 集合
        System.out.println("====== Set ======");
        key = "set:test";
        String key1 = "set:test1";
        String key2 = "set:test2";
        redisTemplate.delete(key1);
        redisTemplate.delete(key2);
        redisTemplate.opsForSet().add(key1, "value1", "value2", "value1-1", "value1-2");
        System.out.println(redisTemplate.opsForSet().members(key1));
        redisTemplate.opsForSet().remove(key1, "value1-1");
        redisTemplate.opsForSet().move(key1, "value1", key2);

        redisTemplate.opsForSet().add(key2, "value2");
        System.out.println(redisTemplate.opsForSet().intersect(key1, key2));// 交集

        redisTemplate.opsForSet().intersectAndStore(key1, key2, key);// 交集
        System.out.println(redisTemplate.opsForSet().members(key));

        System.out.println(redisTemplate.opsForSet().union(key1, key2));// 并集

        System.out.println(redisTemplate.opsForSet().difference(key1, Arrays.asList(key2)));// 差集


        // zset (sorted set): 有序集合
        // 与 Set 类似，略
    }
}
```

![](https://oss.x8y.cc/blog-img/2068/1.png)

# 更进一步

上文中我们与 Redis 交互使用的是 Spring 封装的 StringRedisTemplate, 源码：

```
public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        this.setKeySerializer(RedisSerializer.string());
        this.setValueSerializer(RedisSerializer.string());
        this.setHashKeySerializer(RedisSerializer.string());
        this.setHashValueSerializer(RedisSerializer.string());
    }

    public StringRedisTemplate(RedisConnectionFactory connectionFactory) {
        this();
        this.setConnectionFactory(connectionFactory);
        this.afterPropertiesSet();
    }

    protected RedisConnection preProcessConnection(RedisConnection connection, boolean existingConnection) {
        return new DefaultStringRedisConnection(connection);
    }
}
```

可以看到 StringRedisTemplate 继承自 RedisTemplate 并在构造函数中设置了 key 和 value 的 Serializer (RedisTemplate 默认使用的是 JdkSerializationRedisSerializer, 在 RedisTemplate 源码中可以看到)

RedisSerializer 源码：

```
public interface RedisSerializer<T> {
    @Nullable
    byte[] serialize(@Nullable T var1) throws SerializationException;

    @Nullable
    T deserialize(@Nullable byte[] var1) throws SerializationException;

    static RedisSerializer<Object> java() {
        return java((ClassLoader)null);
    }

    static RedisSerializer<Object> java(@Nullable ClassLoader classLoader) {
        return new JdkSerializationRedisSerializer(classLoader);
    }

    static RedisSerializer<Object> json() {
        return new GenericJackson2JsonRedisSerializer();
    }

    static RedisSerializer<String> string() {
        return StringRedisSerializer.UTF_8;
    }
}

```

Spring 同时也封装了几种 Serializer, 如 JdkSerializationRedisSerializer, GenericJackson2JsonRedisSerializer, StringRedisSerializer 等，详见下图：

![](https://oss.x8y.cc/blog-img/2068/2.png)

# 自定义 RedisTemplate

## 添加依赖

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

## 配置 Serializer

下面代码分别使用了 GenericJackson2JsonRedisSerializer 和 Jackson2JsonRedisSerializer

```
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory){

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());

        // 使用 GenericJackson2JsonRedisSerializer
        template.setValueSerializer(RedisSerializer.json());
        template.setHashValueSerializer(RedisSerializer.json());


        // 使用 Jackson2JsonRedisSerializer
//        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
//        ObjectMapper objectMapper = new ObjectMapper();
//        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
//        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
//
//        template.setValueSerializer(jackson2JsonRedisSerializer);
//        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        return template;

    }
}
```

## 测试

```
@Component
public class ObjectRunner implements CommandLineRunner {

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void run(String... args) throws Exception {

        String key;

        System.out.println("====== Object ======");
        User user = new User();
        user.setId(1);
        user.setName("admin");
        key = "user:id:" + user.getId();
        redisTemplate.opsForValue().set(key, user);
        user = (User)redisTemplate.opsForValue().get(key);
        System.out.println(user.getName());
    }
}
```

P.S. 用冒号作为分割在 Redis 中是设计 key 的一种不成文原则

源码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/redisdemo)

**本人 C# 转 Java 的 newbie, 如果错误或不足欢迎指正，谢谢**


参考：

+ [springboot 整合redis(自定义RedisTemplate)](https://segmentfault.com/a/1190000018883478)
+ [RedisTemplate中hash类型的使用](https://www.jianshu.com/p/569d19fa356d)
+ [RedisTemplate中set类型的使用](https://www.jianshu.com/p/4024e9b9ba93)
+ [RedisTemplate中zset类型的使用](https://www.jianshu.com/p/11ade65876ee)