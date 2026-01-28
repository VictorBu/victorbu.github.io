---
title: Spring Boot 自带缓存及结合 Redis 使用
date: 2019-05-16 17:00:00
updated: 2019-05-16 17:00:00
categories: [IT]
tags: [Java, Spring Boot, Redis]
---

> 本文测试环境: Spring Boot 2.1.4.RELEASE + Redis 5.0.4 + CentOS 7

# 自带缓存

如果没有使用缓存中间件，Spring Boot 会使用默认的缓存，我们只需启用即可

## 在启动类添加 @EnableCaching 注解

```
@SpringBootApplication
@EnableCaching
public class CacheredisApplication {

    public static void main(String[] args) {
        SpringApplication.run(CacheredisApplication.class, args);
    }

}
```

## 缓存配置

+ @Cacheable: 先判断有没有缓存，有缓存取缓存，否则执行后续操作后结果存入缓存
+ @CachePut: 操作后结果存入缓存
+ @CacheEvict: 清除缓存

具体使用见下面示例：

```
@Service
//@CacheConfig(cacheNames = "user") // 如果使用该注解, 方法中则可以省略 cacheNames 配置
public class UserServiceImpl implements UserService {

    @Autowired
    UserDao userDao;

    @Override
    // 缓存的最终 key 值为 user::id
    @Cacheable(cacheNames = "user", key = "#id")
    public User get(int id) {
        return userDao.get(id);
    }

    @Override
    // condition: 执行方法前判断是否使用注解的功能; unless: 执行方法后，判断是否使用注解提供的功能
    @CachePut(cacheNames = "user", key = "#user.id", condition = "#user.id<10", unless = "#result.status = 1")
    public User update(User user) {
        return userDao.update(user);
    }

    @Override
    // 默认规则: 只有一个参数则 key 值取该参数, 如果有多个参数则将这些参数拼接起来作为 key
    @CacheEvict(cacheNames = "user")
    public boolean delete(int id) {
        return userDao.delete(id);
    }

    @Override
    // allEntries 清除 cacheNames 下所有 key; beforeInvocation 方法执行前清除
    @CacheEvict(cacheNames = "user", allEntries = true, beforeInvocation = true)
    public boolean deleteAll() {
        return userDao.deleteAll();
    }
}
```

# 结合 Redis 使用

关于 Redis 基本使用，参考昨天的随笔 [Spring Boot + Redis 初体验](/2019/05/2068-spring-boot-and-redis/)

## 添加配置

```
spring:
  cache:
    type: redis # 设置使用 redis 作为缓存 (此行可以不配置)
    redis:
      time-to-live: 300s
#      key-prefix: key # 不要配置该项
#      use-key-prefix: true # 不要配置该项

  redis:
    host: 192.168.30.101
    port: 6379
    database: 0
#    password: ******
```

## 缓存类添加代码

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    StringRedisTemplate redisTemplate;

    ... ...
}
```

## 注意事项

要缓存的类要实现 Serializable 接口

## 存在问题 (坑)

在配置文件配置 key-prefix 和 use-key-prefix 项生成的 key 会有问题:

|  |  |  |  |
| ------ | ------ | ------ | ------ |
| key-prefix | 不配置 | key- | key- |
| use-key-prefix | 不配置 | true | false |
| Redis缓存内的key | user::1 | key-1 | 1 |

如上所示，如果配置了 key-prefix 和 use-key-prefix 设置的 cacheNames 会被覆盖掉，两个或以上类的对象缓存会有问题

# 未解决问题

结合 Redis 使用时即使使用自定义 RedisTemplate 改变了 Serializer, 但在实际序列化时仍然使用的是默认的 JdkSerializationRedisSerializer，不知道为什么会这样 (这应该也是需要缓存的类为什么必须实现 Serializable 接口)，还恳请大神指教！


源码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/cacheredis)

**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**


参考：

[用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034)
