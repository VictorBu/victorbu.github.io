---
title: Spring Boot 内置定时任务
date: 2019-04-30 17:00:00
updated: 2019-04-30 17:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

# 启用定时任务

```
@SpringBootApplication
@EnableScheduling // 启动类添加 @EnableScheduling 注解
public class ScheduleDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ScheduleDemoApplication.class, args);
    }

}
```

# 新增定时任务类

```
@Component // 类上添加 @Component 注解
public class TaskDemo {

    private static final Logger logger = LoggerFactory.getLogger(TaskDemo.class);

    @Scheduled(cron = "0/5 * * * * ? ") // 方法上添加 @Scheduled 注解
    public void job1(){
        try {
            logger.info("job1");
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Scheduled(cron = "0/5 * * * * ? ")
    public void job2(){
        try {
            logger.info("job2");
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

![](https://oss.x8y.cc/blog-img/2065/1.PNG)

# 多线程执行

从上面图片可以看到开启多个任务是以单线程执行的，执行完当前任务才会继续执行下一个

启用多线程执行有两种方式：

## 使用默认线程池

```
@Component
@EnableAsync // 类上添加 @EnableAsync 注解
public class TaskDemo {

    ... ...

    @Async // 方法上添加 @Async 注解
    @Scheduled(cron = "0/5 * * * * ? ")
    public void job1(){
        ... ...
    }

    ... ...
}
```

![](https://oss.x8y.cc/blog-img/2065/2.PNG)

## 使用自定义线程池

添加配置类：

```
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();

        threadPoolTaskScheduler.setPoolSize(2);
        threadPoolTaskScheduler.setThreadNamePrefix("my-pool-");
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
```

![](https://oss.x8y.cc/blog-img/2065/3.PNG)

> 参考

+ [springboot + @scheduled 多任务并发](https://www.cnblogs.com/shamo89/p/8341400.html)
+ [How to Schedule Tasks with Spring Boot](https://www.callicoder.com/spring-boot-task-scheduling-with-scheduled-annotation/)
