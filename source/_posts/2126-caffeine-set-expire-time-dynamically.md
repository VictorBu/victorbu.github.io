---
title: Caffeine 动态设置过期时间
date: 2023-06-16 10:00:00
updated: 2023-06-16 10:00:00
categories: [IT]
tags: [Cache, Caffeine, Java]
---

# 实现 Expiry 接口

```
public class CaffeineExpiry implements Expiry<String, Object> {
    @Override
    public long expireAfterCreate(@NonNull String key, @NonNull Object value, long currentTime) {
        return 0;
    }

    @Override
    public long expireAfterUpdate(@NonNull String key, @NonNull Object value, long currentTime, @NonNegative long currentDuration) {
        return currentDuration;
    }

    @Override
    public long expireAfterRead(@NonNull String key, @NonNull Object value, long currentTime, @NonNegative long currentDuration) {
        return currentDuration;
    }
}
```

注意：如果使用常规的 put(key, value) 方法，则需要 expireAfterCreate 方法返回具体的值；下文使用 put(key, value, duration, timeUnit) 方法，expireAfterCreate 方法可以范围任意值，不影响缓存过期时间。

# 配置

```
@Configuration
public class CaffeineConfig {
    @Bean
    public Cache<String, Object> caffeineCache() {
        return Caffeine.newBuilder()
                .expireAfter(new CaffeineExpiry())
                .initialCapacity(100)
                .maximumSize(1000)
                .build();
    }
}
```

# 使用

```
@Component
public class CaffeineCacheService {
    @Autowired
    Cache<String, Object> caffeineCache;

    public void set(String key, Object value, long duration, TimeUnit unit) {
        caffeineCache.policy().expireVariably().ifPresent(e->{
            e.put(key, value, duration, unit);
        });
    }

    public <T> T get(String key) {
        return (T)caffeineCache.getIfPresent(key);
    }
}
```

> 参考

[Eviction#time-based](https://github.com/ben-manes/caffeine/wiki/Eviction#time-based)
[解读JVM级别本地缓存Caffeine青出于蓝的要诀3 —— 讲透Caffeine的数据驱逐淘汰机制与用法](https://www.cnblogs.com/softwarearch/p/16927947.html)
[ Spring 缓存动态设置过期时间 - Caffeine](https://stackoom.com/question/3al5t)