---
title: DotNetty 实现 Modbus TCP 系列 (三) Codecs & Handler
date: 2019-02-14 08:30:00
updated: 2019-02-14 08:30:00
categories: [IT]
tags: [DotNetty, Modbus]
---

DotNetty 作为一个半成品，我们不需要关注细节的实现，只需要关注自己的业务即可，所以最主要的就是处理 Codecs 和 Handler。

所有的 Codecs 和 Handler 均直接或间接继承自 ChannelHandlerAdapter。为什么要分为 Codecs 和 Handler，个人理解是 Codecs 负责将消息解码为我们所需的对象或者将处理的结果编码，Handler 对解码得到的对象进行逻辑处理，达到职责分离的目的。

DotNetty 中可以注册多个 Codecs/Handler，入站消息按照注册的先后顺序执行，出站消息按照注册的先后逆序执行。

对于 Client 端：

+ 入站：ModbusDecoder --> ModbusResponseHandler
+ 出站：ModbusEncoder

对于 Server 端：

+ 入站：ModbusDecoder --> ModbusRequestHandler
+ 出站：ModbusEncoder

# ModbusDecoder

```C#
public class ModbusDecoder : ByteToMessageDecoder
{
	private bool isServerMode;
	private readonly short maxFunctionCode = 0x80;
	private readonly string typeName = "Karonda.ModbusTcp.Entity.Function.{0}.{1}{0}";

	public ModbusDecoder(bool isServerMode)
	{
		this.isServerMode = isServerMode;
	}
	protected override void Decode(IChannelHandlerContext context, IByteBuffer input, List<object> output)
	{
		//Transaction Identifier + Protocol Identifier + Length + Unit Identifier + Function Code
		if (input.Capacity < 2 + 2 + 2 + 1 + 1)
		{
			return;
		}

		ModbusHeader header = new ModbusHeader(input);
		short functionCode = input.ReadByte();
		ModbusFunction function = null;

		if(Enum.IsDefined(typeof(ModbusCommand), functionCode))
		{
			var command = Enum.GetName(typeof(ModbusCommand), functionCode);

			function = (ModbusFunction)Activator.CreateInstance(Type.GetType(string.Format(typeName, isServerMode ? "Request" : "Response", command)));
		}


		if (functionCode >= maxFunctionCode)
		{
			function = new ExceptionFunction(functionCode);
		}
		else if(function == null)
		{
			function = new ExceptionFunction(functionCode, 0x01);
		}

		function.Decode(input);
		ModbusFrame frame = new ModbusFrame(header, function);

		output.Add(frame);
	}
}
```

ModbusDecoder 继承了 ByteToMessageDecoder。继承了 ByteToMessageDecoder 的类必须实现的唯一的抽象方法：Decode，该方法将 ByteBuffer 解析为 List，如果 List 不为空则会将该 List 传递给下一个 ChannelHandlerAdapter。

ModbusDecoder 同时为 Client 端和 Server 端使用，如果是 Server 端则将消息解析成请求类，反之如果是 Client 端则将消息解析成响应类。

# ModbusResponseHandler

```C#
public class ModbusResponseHandler : SimpleChannelInboundHandler<ModbusFrame>
{
	private Dictionary<ushort, ModbusFrame> responses = new Dictionary<ushort, ModbusFrame>();
	protected override void ChannelRead0(IChannelHandlerContext ctx, ModbusFrame msg)
	{
		responses.Add(msg.Header.TransactionIdentifier, msg);
	}

	public override void ExceptionCaught(IChannelHandlerContext context, Exception exception)
	{
		context.CloseAsync();
	}
}
```

将接收到的响应信息加入 responses 供后续处理。

# ModbusRequestHandler

```C#
public class ModbusRequestHandler : SimpleChannelInboundHandler<ModbusFrame>
{
	private ModbusResponseService responseService;
	public ModbusRequestHandler(ModbusResponseService responseService)
	{
		this.responseService = responseService;
	}

	protected override void ChannelRead0(IChannelHandlerContext ctx, ModbusFrame msg)
	{
		var function = msg.Function;
		var  response = responseService.Execute(function);

		var header = msg.Header;
		var frame = new ModbusFrame(header, response);

		ctx.WriteAndFlushAsync(frame);
	}
}
```

responseService 为一个抽象类，用来自定义处理接收到的请求并返回结果，需要在实现 Server 端时继承并实现。

```C#
public abstract class ModbusResponseService
{
	public ModbusFunction Execute(ModbusFunction function)
	{
		if (function is ReadHoldingRegistersRequest)
		{
			var request = (ReadHoldingRegistersRequest)function;
			return ReadHoldingRegisters(request);
		}

		throw new Exception("Function Not Support");
	}

	public abstract ModbusFunction ReadHoldingRegisters(ReadHoldingRegistersRequest request);
}
```

(文中代码仅添加了 0x03 的方法)

# ModbusEncoder

```C#
public class ModbusEncoder : ChannelHandlerAdapter
{
	public override Task WriteAsync(IChannelHandlerContext context, object message)
	{
		if (message is ModbusFrame)
		{
			var frame = (ModbusFrame)message;
			return context.WriteAndFlushAsync(frame.Encode());
		}

		return context.WriteAsync(message);
	}
}
```

如果是 ModbusFrame 消息则 Flush，否则传递到下一个 ChannelHandlerAdapter。

开源地址：[modbus-tcp](https://github.com/VictorBu/modbus-tcp)