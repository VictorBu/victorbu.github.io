---
title: Netty 客户端断线重连
date: 2019-04-29 17:00:00
updated: 2019-04-29 17:00:00
categories: [IT]
tags: [Java, Netty]
---

> client 关闭后会执行 finally 代码块，可以在这里可以进行重连操作

```
public class NettyClient implements Runnable {

    private final String host;
    private final int port;
    private final int reconnectSleepSeconds;

    public NettyClient(String host, int port, int reconnectSleepSeconds){
        this.host = host;
        this.port = port;
        this.reconnectSleepSeconds = reconnectSleepSeconds;
    }

    @Override
    public void run() {
        connect();
    }

    private void connect(){
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            Bootstrap b = new Bootstrap();
            b.group(workerGroup);
            b.channel(NioSocketChannel.class);
            b.option(ChannelOption.SO_KEEPALIVE, true);
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {

                    // TODO: 添加 Handler
                }
            });

            ChannelFuture f = b.connect(host, port).sync();

            f.channel().closeFuture().sync();
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            workerGroup.shutdownGracefully();

            try {
                TimeUnit.SECONDS.sleep(reconnectSleepSeconds);

                connect(); // 断线重连

            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}
```


参考：[微言netty：不在浮沙筑高台](https://www.cnblogs.com/scy251147/p/10498008.html)
