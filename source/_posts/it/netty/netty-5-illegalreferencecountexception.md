---
title: Netty 5 io.netty.util.IllegalReferenceCountException 异常
date: 2019-04-02 17:00:00
updated: 2019-04-02 17:00:00
categories: [IT]
tags: [Java, Netty]
---


# 异常信息

io.netty.util.IllegalReferenceCountException: refCnt: 0, decrement: 1

# 原因

handler 继承了 SimpleChannelInboundHandler，SimpleChannelInboundHandler 中 channelRead 代码如下：

```
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    boolean release = true;

    try {
        if (this.acceptInboundMessage(msg)) {
            this.messageReceived(ctx, msg);
        } else {
            release = false;
            ctx.fireChannelRead(msg);
        }
    } finally {
        if (this.autoRelease && release) {
            ReferenceCountUtil.release(msg);
        }

    }

}
```

在 finally 代码块中释放了 msg

# 解决方案

handler 改继承 ChannelHandlerAdapter


> 参考：[netty 中遇到的一个坑 SimpleInboundHandler，记录一下](https://my.oschina.net/rpgmakervx/blog/687190)