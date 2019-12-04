---
title: Netty 搭建 WebSocket 服务端
date: 2019-12-04 09:30:00
updated: 2019-12-04 09:30:00
categories: [IT]
tags: [Java, Netty, WebSocket]
---

# 一、编码器、解码器

```
... ...

@Autowired
private HttpRequestHandler httpRequestHandler;
@Autowired
private TextWebSocketFrameHandler textWebSocketFrameHandler;

... ...

.childHandler(new ChannelInitializer<SocketChannel> () {

	@Override
	protected void initChannel(SocketChannel channel) throws Exception {
		// WebSocket 是基于 Http 协议的，要使用 Http 解编码器
		channel.pipeline().addLast("http-codec", new HttpServerCodec());
		// 用于大数据流的分区传输
		channel.pipeline().addLast("http-chunked",new ChunkedWriteHandler());
		// 将多个消息转换为单一的 request 或者 response 对象，最终得到的是 FullHttpRequest 对象
		channel.pipeline().addLast("aggregator", new HttpObjectAggregator(65536));
		// 创建 WebSocket 之前会有唯一一次 Http 请求 (Header 中包含 Upgrade 并且值为 websocket)
		channel.pipeline().addLast("http-request",httpRequestHandler);
		// 处理所有委托管理的 WebSocket 帧类型以及握手本身
		// 入参是 ws://server:port/context_path 中的 contex_path
		channel.pipeline().addLast("websocket-server", new WebSocketServerProtocolHandler(socketUri));
		// WebSocket RFC 定义了 6 种帧，TextWebSocketFrame 是我们唯一真正需要处理的帧类型
		channel.pipeline().addLast("text-frame",textWebSocketFrameHandler);
	}
});

... ...

```

其中 HttpRequestHandler 和 TextWebSocketFrameHandler 是自定义 Handler


## 1.1 HttpRequestHandler

```
@Component
@ChannelHandler.Sharable
public class HttpRequestHandler extends SimpleChannelInboundHandler<FullHttpRequest> {

    private static final Logger LOGGER = LoggerFactory.getLogger(HttpRequestHandler.class);

    @Value("${server.socket-uri}")
    private String socketUri;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest msg) throws Exception {
        if (msg.uri().startsWith(socketUri)) {
            String userId = UriUtil.getParam(msg.uri(), "userId");
            if (userId != null) {
                // todo: 用户校验，重复登录判断
                ChannelSupervise.addChannel(userId, ctx.channel());
                ctx.fireChannelRead(msg.setUri(socketUri).retain());
            } else {
                ctx.close();
            }
        } else {
            ctx.close();
        }
    }

}
```

## 1.2 TextWebSocketFrameHandler

```
@Component
@ChannelHandler.Sharable
public class TextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    private static final Logger LOGGER = LoggerFactory.getLogger(TextWebSocketFrameHandler.class);

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        if (evt instanceof WebSocketServerProtocolHandler.HandshakeComplete) {
            ctx.pipeline().remove(HttpRequestHandler.class);
        }
        super.userEventTriggered(ctx, evt);
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        String requestMsg = msg.text();
        String responseMsg = "服务端接收客户端消息：" + requestMsg;
        TextWebSocketFrame resp = new TextWebSocketFrame(responseMsg);
        ctx.writeAndFlush(resp.retain());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
        LOGGER.error(ctx.channel().id().asShortText(), cause);
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception { // (6)
        ChannelSupervise.removeChannel(ctx.channel());
        LOGGER.info("[%s]断开连接", ctx.channel().id().asShortText());
    }
}
```

# 二、主动向客户端推送消息

## 2.1 推送工具类

```
public class ChannelSupervise {

    private static ChannelGroup GlobalGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    private static ConcurrentMap<String, ChannelId> UserChannelMap = new ConcurrentHashMap();
    private static ConcurrentMap<String, String> ChannelUserMap = new ConcurrentHashMap();

    public static void addChannel(String userId, Channel channel){
        GlobalGroup.add(channel);
        UserChannelMap.put(userId, channel.id());
        ChannelUserMap.put(channel.id().asShortText(), userId);
    }
    public static void removeChannel(Channel channel){
        GlobalGroup.remove(channel);
        String userId = ChannelUserMap.get(channel.id().asShortText());
        UserChannelMap.remove(userId);
        ChannelUserMap.remove(channel.id().asShortText());
    }
    public static void sendToUser(String userId, String msg){
        TextWebSocketFrame textWebSocketFrame = new TextWebSocketFrame(msg);
        Channel channel = GlobalGroup.find(UserChannelMap.get(userId));
        channel.writeAndFlush(textWebSocketFrame);
    }
    public static void sendToAll(String msg){
        TextWebSocketFrame textWebSocketFrame = new TextWebSocketFrame(msg);
        GlobalGroup.writeAndFlush(textWebSocketFrame);
    }
}
```

支持向具体某个客户端发送消息，或者群发消息

## 2.2 推送接口

```
@RestController
public class WebsocketController {
    @RequestMapping("sendToAll")
    public void sendToAll(String msg) {
        ChannelSupervise.sendToAll(msg);
    }

    @RequestMapping("sendToUser")
    public void sendToUser(String userId, String msg) {
        ChannelSupervise.sendToUser(userId, msg);
    }
}
```

# 三、测试

```
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>WebSocket客户端</title>
	</head>
	<body>
	
		<script type="text/javascript">
		
			var socket;
			
			function connect(){
				var userId = document.getElementById('userId').value;
				if(window.WebSocket){
					// 参数就是与服务器连接的地址
					// socket = new WebSocket('ws://localhost:8081/ws');
					socket = new WebSocket('ws://localhost:8081/ws?userId=' + userId);
					
					
					// 客户端收到服务器消息的时候就会执行这个回调方法
					socket.onmessage = function (event) {
						var response = document.getElementById('response');
						response.innerHTML = response.innerHTML 
							+ '<p style="color:LimeGreen;"> 接收：' + event.data + '</p>';
					}

					// 连接建立的回调函数
					socket.onopen = function(event){
						var status = document.getElementById('status');
						status.innerHTML = '<p style="color:YellowGreen;">WebSocket 连接开启</p>';
					}

					// 连接断掉的回调函数
					socket.onclose = function (event) {
						var status = document.getElementById('status');
						status.innerHTML = '<p style="color:Red;">WebSocket 连接关闭</p>';
					}
				}else{
					var status = document.getElementById('status');
					status.innerHTML = '<p style="color:Red;">浏览器不支持 WebSocket</p>';
				}
			}

			// 发送数据
			function send(message){
				if(!window.WebSocket){
					return;
				}
				
				var ta = document.getElementById('response');
				ta.innerHTML = ta.innerHTML + '<p style="color:SkyBlue;"> 发送：' + message + '</p>';

				// 当websocket状态打开
				if(socket.readyState == WebSocket.OPEN){
					socket.send(message);
				}else{
					var response = document.getElementById("response");
					response.innerHTML = '<p style="color:Red;">连接没有开启</p>';
				}
			}
		</script>
		
		<form onsubmit="return false">
			<label for="userId">用户ID:</label>
			<input type="text" name="userId" id="userId" />
			<input type ="button" value="连接服务器" onclick="connect();">
		</form>
		
		<div id ="status"></div>
		
		<form onsubmit="return false">
			<input name = "message" style="width: 200px;"></input>
			<input type ="button" value="发送消息" onclick="send(this.form.message.value);">
		</form>
		
		<div id ="response"></div>
		
		<input type="button" onclick="javascript:document.getElementById('response').innerHTML=''" value="清空消息">
	</body>
</html>
```

> 注意

因为自定义 Handler 使用依赖注入实例化，所以需要添加 @ChannelHandler.Sharable 注解，否则会报错：is not a @Sharable handler, so can't be added or removed multiple times.

> 参考

1. [微言Netty：分布式服务框架](https://www.cnblogs.com/scy251147/p/10983814.html)
1. [基于netty搭建websocket，实现消息的主动推送](https://www.jianshu.com/p/56216d1052d7)
1. [Netty笔记之六：Netty对websocket的支持](https://www.jianshu.com/p/9a97e667cf84)
1. [使用Netty做WebSocket服务端](https://www.cnblogs.com/maybo/p/5600154.html)

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/netty-websocket-server)

