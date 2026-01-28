---
title: Spring Boot + Netty 中 @Autowired, @Value 为空解决
date: 2019-04-11 17:00:00
updated: 2019-04-11 17:00:00
categories: [IT]
tags: [Java, Spring Boot, Netty]
---

# 问题描述

使用 Spring Boot + Netty 新建项目时 Handler 中的 @Autowired, @Value 注解的始终为空值

# 解决方法


```
@Component // 1. 添加 @Component 注解
public class TestHandler extends ChannelInboundHandlerAdapter {

    private static TestHandler testHandler; // 2. 定义本类的静态对象

    @Autowired
    TestService testService;

    @PostConstruct // 3. 添加 @PostConstruct 注解的方法
    public void init(){
        testHandler = this;
    }


    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {

        ByteBuf buf = (ByteBuf) msg;

        byte[] bytes = new byte[buf.readableBytes()];
        buf.readBytes(bytes);

        testHandler.testService.handlerMessage(bytes); // 4. 使用

    }

}
```

参考：[Netty handler处理类无法使用@Autowired注入bean的解决方法](http://www.mamicode.com/info-detail-2352016.html)