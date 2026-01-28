---
title: Netty 心跳处理
date: 2019-12-08 09:30:00
updated: 2019-12-08 09:30:00
categories: [IT]
tags: [Java, Netty]
---

> 传统的心跳包设计，基本上是服务端和客户端同时维护 Scheduler，然后双方互相接收心跳包信息，然后重置双方的上下线状态表。此种心跳方式的设计，可以选择在主线程上进行，也可以选择在心跳线程中进行，由于在进行业务调用过程中，此种心跳包是没有必要进行发送的，所以在一定程度上会造成资源浪费。严重的甚至会影响业务线程的操作。但是在 Netty 中是通过检测链路的空闲与否在进行的。链路分为读操作空闲，写操作空闲，读写操作空闲。由于空闲检测本身只有在通道空闲的时候才进行检测，而不是固定频率的进行心跳包通讯，所以可以节省网络带宽，同时对业务的影响也很小

在 Netty 中空闲检测需要引入 IdleStateHandler，然后实现自己的心跳处理 Handler，本文中服务端与客户端均向对方发送心跳包。

# 一、服务端

## 1.1 编解码及 Handler

```
...

.childHandler(new ChannelInitializer<SocketChannel>() {

	@Override
	protected void initChannel(SocketChannel channel) throws Exception {
		channel.pipeline().addLast("ping", new IdleStateHandler(10, 5, 10));
		channel.pipeline().addLast("encoder", new NettyMessageEncoder());
		channel.pipeline().addLast("decoder", new NettyMessageDecoder());
		channel.pipeline().addLast("message", new MessageHandler());
		channel.pipeline().addLast("heartbeat", new HeartbeatHandler());
	}
});

...

```

HeartbeatHandler 为 心跳处理 Handler

## 1.2 心跳处理 Handler

```
public class HeartbeatHandler extends ChannelInboundHandlerAdapter {
    private static final Logger LOGGER = LoggerFactory.getLogger(HeartbeatHandler.class);

    private final AttributeKey<Integer> counterAttr = AttributeKey.valueOf(ChannelSupervise.COUNTER_ATTR);;

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        super.userEventTriggered(ctx, evt);

        if(evt instanceof IdleStateEvent) {
            IdleStateEvent idleStateEvent = (IdleStateEvent)evt;
            switch (idleStateEvent.state()) {
                case READER_IDLE:
                    NettyMessage<String> nettyMessage = new NettyMessage<>();
                    nettyMessage.setSessionId(0L);
                    nettyMessage.setType(NettyMessageTypeEnum.HEARTBEAT);

                    ctx.writeAndFlush(nettyMessage).addListener(future -> {
//                        if(future.isSuccess()) {
//                            ctx.channel().attr(counterAttr).set(0);
//                        }else {
                            Integer counter = ctx.channel().attr(counterAttr).get();
                            counter = counter + 1;
                            LOGGER.info(ctx.channel().id().asShortText() + "，发送心跳: " + counter);
                            if(counter >= 3) {
                                ctx.close();
                            } else {
                                ctx.channel().attr(counterAttr).set(counter);
                            }
//                        }
                    });
                    break;
                default:
                    break;
            }
        }
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.channel().attr(counterAttr).set(0);
        ChannelSupervise.addChannel(ctx.channel());
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        ChannelSupervise.removeChannel(ctx.channel());
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        ctx.channel().attr(counterAttr).set(0);
        ctx.fireChannelRead(msg);
    }



    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        LOGGER.error("断开连接", cause);
        ctx.close();
    }
}
```

本例中，如果服务端连续发送三次心跳包，则认为客户端断开连接，使用 Netty 内置的 AttributeKey 计数 (本例中为方便测试注释掉部分代码，正常来说如果发送消息成功则证明客户端还在线，需要把计数重置为 0)。

# 二、客户端

## 2.1 编解码及 Handler

```
...

.handler(new ChannelInitializer<SocketChannel>() {

	@Override
	protected void initChannel(SocketChannel channel) throws Exception {
		channel.pipeline().addLast("ping", new IdleStateHandler(5, 5, 3));
		channel.pipeline().addLast("encoder", new NettyMessageEncoder());
		channel.pipeline().addLast("decoder", new NettyMessageDecoder());
		channel.pipeline().addLast("heartbeat", new HeartbeatHandler());
		channel.pipeline().addLast("logger", new LoggingHandler(LogLevel.INFO));
	}
});

...
```

HeartbeatHandler 为 心跳处理 Handler

## 2.2 心跳处理 Handler

```
public class HeartbeatHandler extends ChannelInboundHandlerAdapter {
    private static final Logger LOGGER = LoggerFactory.getLogger(HeartbeatHandler.class);

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {

        if(evt instanceof IdleStateEvent) {
            IdleStateEvent idleStateEvent = (IdleStateEvent) evt;

            switch (idleStateEvent.state()){
                case WRITER_IDLE:
                    LOGGER.info("发送心跳包");
                    NettyMessage<String> nettyMessage = new NettyMessage<>();
                    nettyMessage.setSessionId(0L);
                    nettyMessage.setType(NettyMessageTypeEnum.HEARTBEAT);
                    ctx.writeAndFlush(nettyMessage).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
                    break;
                default:
                    break;
            }
        }

        super.userEventTriggered(ctx, evt);
    }

    ...
}
```

本例中，如果客户端发送心跳消息失败则断开连接。

> 参考

1. [Netty(一) SpringBoot 整合长连接心跳机制](https://crossoverjie.top/2018/05/24/netty/Netty(1)TCP-Heartbeat/)
1. [微言Netty：分布式服务框架](https://www.cnblogs.com/scy251147/p/10983814.html)


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/netty-heartbeat)

