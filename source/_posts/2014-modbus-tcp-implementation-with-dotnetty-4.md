---
title: DotNetty 实现 Modbus TCP 系列 (四) Client & Server
date: 2019-02-14 14:52:00
updated: 2019-02-14 14:52:00
categories: [IT]
tags: [DotNetty, Modbus]
---

# Client

```C#
public class ModbusClient
{
	public string Ip { get; }
	public int Port { get; }
	public short UnitIdentifier { get; }
	public IChannel Channel { get; private set; }

	private MultithreadEventLoopGroup group;
	private ConnectionState connectionState;
	private ushort transactionIdentifier;
	private readonly string handlerName = "response";

	public ModbusClient(short unitIdentifier, string ip, int port = 502)
	{
		Ip = ip;
		Port = port;
		UnitIdentifier = unitIdentifier;

		connectionState = ConnectionState.NotConnected;
	}

	public async Task Connect()
	{
		group = new MultithreadEventLoopGroup();

		try
		{
			var bootstrap = new Bootstrap();
			bootstrap
				.Group(group)
				.Channel<TcpSocketChannel>()
				.Option(ChannelOption.TcpNodelay, true)
				.Handler(new ActionChannelInitializer<ISocketChannel>(channel =>
				{
					IChannelPipeline pipeline = channel.Pipeline;

					pipeline.AddLast("encoder", new ModbusEncoder());
					pipeline.AddLast("decoder", new ModbusDecoder(false));

					pipeline.AddLast(handlerName, new ModbusResponseHandler());
				}));

			connectionState = ConnectionState.Pending;

			Channel = await bootstrap.ConnectAsync(new IPEndPoint(IPAddress.Parse(Ip), Port));

			connectionState = ConnectionState.Connected;
		}
		catch (Exception exception)
		{
			throw exception;
		}
	}

	public async Task Close()
	{
		if (ConnectionState.Connected == connectionState)
		{
			try
			{
				await Channel.CloseAsync();
			}
			finally
			{
				await group.ShutdownGracefullyAsync(TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(1));

				connectionState = ConnectionState.NotConnected;
			}
		}
	}

	public ushort CallModbusFunction(ModbusFunction function)
	{
		if (ConnectionState.Connected != connectionState || Channel == null)
		{
			throw new Exception("Not connected!");
		}

		SetTransactionIdentifier();

		ModbusHeader header = new ModbusHeader(transactionIdentifier, UnitIdentifier);
		ModbusFrame frame = new ModbusFrame(header, function);
		Channel.WriteAndFlushAsync(frame);

		return transactionIdentifier;
	}

	public T CallModbusFunctionSync<T>(ModbusFunction function) where T : ModbusFunction
	{
		var transactionIdentifier = CallModbusFunction(function);

		var handler = (ModbusResponseHandler)Channel.Pipeline.Get(handlerName);
		if (handler == null)
		{
			throw new Exception("Not connected!");
		}

		return (T)handler.GetResponse(transactionIdentifier).Function;
	}

	private void SetTransactionIdentifier()
	{
		if (transactionIdentifier < ushort.MaxValue)
		{
			transactionIdentifier++;
		}
		else
		{
			transactionIdentifier = 1;
		}
	}

	public ushort ReadHoldingRegistersAsync(ushort startingAddress, ushort quantity)
	{
		var function = new ReadHoldingRegistersRequest(startingAddress, quantity);
		return CallModbusFunction(function);
	}

	public ReadHoldingRegistersResponse ReadHoldingRegisters(ushort startingAddress, ushort quantity)
	{
		var function = new ReadHoldingRegistersRequest(startingAddress, quantity);
		return CallModbusFunctionSync<ReadHoldingRegistersResponse>(function);
	}
}



public enum ConnectionState
{
	NotConnected = 0,
	Connected = 1,
	Pending = 2,
}
```

(文中代码仅添加了 0x03 的方法)

在 Client 中封装了 Modbus 请求方法，对同一个功能同时有同步方法(ReadHoldingRegistersAsync)和异步方法(ReadHoldingRegisters)。同步方法仅返回 TransactionIdentifier(传输标识)，异步方法返回响应结果。

ModbusResponseHandler 修改为：

```C#
public class ModbusResponseHandler : SimpleChannelInboundHandler<ModbusFrame>
{
	private readonly int timeoutMilliseconds = 2000;
	private Dictionary<ushort, ModbusFrame> responses = new Dictionary<ushort, ModbusFrame>();
	protected override void ChannelRead0(IChannelHandlerContext ctx, ModbusFrame msg)
	{
		responses.Add(msg.Header.TransactionIdentifier, msg);
	}

	public ModbusFrame GetResponse(ushort transactionIdentifier)
	{
		ModbusFrame frame = null;
		var timeoutDateTime = DateTime.Now.AddMilliseconds(timeoutMilliseconds);
		do
		{
			Thread.Sleep(1);
			if (responses.ContainsKey(transactionIdentifier))
			{
				frame = responses[transactionIdentifier];
				responses.Remove(transactionIdentifier);
			}
		}
		while (frame == null && DateTime.Now < timeoutDateTime);

		if(frame == null)
		{
			throw new Exception("No Response");
		}
		else if(frame.Function is ExceptionFunction)
		{
			throw new Exception(frame.Function.ToString());
		}

		return frame;
	}

	public override void ExceptionCaught(IChannelHandlerContext context, Exception exception)
	{
		context.CloseAsync();
	}
}
```

# Server

```C#
public class ModbusServer
{
	private ModbusResponseService responseService;
	private ServerState serverState;
	public int Port { get; }
	public IChannel Channel { get; private set; }
	private IEventLoopGroup bossGroup;
	private IEventLoopGroup workerGroup;
	public ModbusServer(ModbusResponseService responseService, int port = 502)
	{
		this.responseService = responseService;
		Port = port;
		serverState = ServerState.NotStarted;
	}

	public async Task Start()
	{
		bossGroup = new MultithreadEventLoopGroup(1);
		workerGroup = new MultithreadEventLoopGroup();

		try
		{
			var bootstrap = new ServerBootstrap();
			bootstrap.Group(bossGroup, workerGroup);

			bootstrap
				.Channel<TcpServerSocketChannel>()
				.Option(ChannelOption.SoBacklog, 100)
				.ChildHandler(new ActionChannelInitializer<IChannel>(channel =>
				{
					IChannelPipeline pipeline = channel.Pipeline;
					pipeline.AddLast("encoder", new ModbusEncoder());
					pipeline.AddLast("decoder", new ModbusDecoder(true));

					pipeline.AddLast("request", new ModbusRequestHandler(responseService));
				}));

			serverState = ServerState.Starting;

			Channel = await bootstrap.BindAsync(Port);

			serverState = ServerState.Started;
		}
		finally
		{
			
		}
	}

	public async Task Stop()
	{
		if (ServerState.Starting == serverState)
		{
			try
			{
				await Channel.CloseAsync();
			}
			finally
			{
				await Task.WhenAll(
					bossGroup.ShutdownGracefullyAsync(TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(1)),
					workerGroup.ShutdownGracefullyAsync(TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(1)));

				serverState = ServerState.NotStarted;
			}
		}
	}
}

public enum ServerState
{
	NotStarted = 0,
	Started = 1,
	Starting = 2,
}
```

实例化 Server 时需要传入 ModbusResponseService 的实现，实现示例：

```C#
public class ModbusResponse : ModbusResponseService
{
	public override ModbusFunction ReadHoldingRegisters(ReadHoldingRegistersRequest request)
	{
		var registers = ReadRegisters(request.Quantity);
		var response = new ReadHoldingRegistersResponse(registers);

		return response;
	}
	
	private ushort[] ReadRegisters(ushort quantity)
	{
		var registers = new ushort[quantity];

		Random ran = new Random();
		for (int i = 0; i < registers.Length; i++)
		{
			registers[i] = (ushort)ran.Next(ushort.MinValue, ushort.MaxValue);
		}

		return registers;
	}
}
```

(文中代码仅添加了 0x03 的方法)

开源地址：[modbus-tcp](https://github.com/VictorBu/modbus-tcp)