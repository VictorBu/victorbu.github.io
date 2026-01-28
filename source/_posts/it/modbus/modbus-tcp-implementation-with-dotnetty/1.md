---
title: DotNetty 实现 Modbus TCP 系列 (一) 报文类
date: 2019-02-13 15:06:00
updated: 2019-02-13 15:06:00
categories: [IT]
tags: [DotNetty, Modbus]
---

# Modbus TCP/IP 报文

![](https://oss.x8y.cc/blog-img/2011/adu.png)

+ 报文最大长度为 260 byte (ADU = 7 byte MBAP Header + 253 byte PDU)
+ Length = Unit Identifier 长度 + PDU 长度

## MBAP Header

![](https://oss.x8y.cc/blog-img/2011/header.png)

## PDU

PDU 由两部分构成：Function Code(功能码) 和 Data 组成


### Function Code

部分功能码：

![](https://oss.x8y.cc/blog-img/2011/functioncode.png)


# 报文类

## ModbusHeader

```C#
public class ModbusHeader
{
	public ushort TransactionIdentifier { get; set; }
	public ushort ProtocolIdentifier { get; set; }
	public ushort Length { get; set; }
	public short UnitIdentifier { get; set; }

	public ModbusHeader(IByteBuffer buffer)
	{
		TransactionIdentifier = buffer.ReadUnsignedShort();
		ProtocolIdentifier = buffer.ReadUnsignedShort();
		Length = buffer.ReadUnsignedShort();
		UnitIdentifier = buffer.ReadByte();
	}

	public ModbusHeader(ushort transactionIdentifier, short unitIdentifier)
		: this(transactionIdentifier, 0x0000, unitIdentifier) // for modbus protocol: Protocol Identifier = 0x00
	{

	}

	private ModbusHeader(ushort transactionIdentifier, ushort protocolIdentifier, short unitIdentifier)
	{
		TransactionIdentifier = transactionIdentifier;
		ProtocolIdentifier = protocolIdentifier;
		UnitIdentifier = unitIdentifier;
	}

	public IByteBuffer Encode()
	{
		IByteBuffer buffer = Unpooled.Buffer();

		buffer.WriteUnsignedShort(TransactionIdentifier);
		buffer.WriteUnsignedShort(ProtocolIdentifier);
		buffer.WriteUnsignedShort(Length);
		buffer.WriteByte(UnitIdentifier);

		return buffer;
	}
}
```

ModbusHeader 对应 MBAP Header，包含两个构造函数：第一个构造函数用于从缓冲区解析消息头，第二个构造函数用来请求/响应时手动构造消息头。Encode 方法用于在传输前对消息头进行编码。

## ModbusFunction

```C#
public abstract class ModbusFunction
{
	protected short FunctionCode { get; }

	protected ModbusFunction(short functionCode)
	{
		FunctionCode = functionCode;
	}
	/// <summary>
	/// PDU length -1 (not include function code length)
	/// </summary>
	/// <returns></returns>
	public abstract int CalculateLength();   

	public abstract void Decode(IByteBuffer buffer);

	public abstract IByteBuffer Encode();

}
```

ModbusFunction 对应 PDU，该类为抽象类，所有的请求/相应的 PDU 均继承自该类。实际使用中根据 FunctionCode 实例化具体的子类对象。其中 CalculateLength 方法用来计算 Data 部分的长度，Decode 方法用于从缓冲区解析 Data，Encode 方法用于在传输前对 Data 编码。

## ModbusFrame

```C#
public class ModbusFrame
{
	public ModbusHeader Header { get; set; }
	public ModbusFunction Function { get; set; }

	public ModbusFrame(ModbusHeader header, ModbusFunction function)
	{
		Header = header;
		Function = function;
	}

	public IByteBuffer Encode()
	{
		Header.Length = (ushort)(1 + 1 + Function.CalculateLength());// Unit Identifier + Function Code + data length

		IByteBuffer buffer = Unpooled.Buffer();

		buffer.WriteBytes(Header.Encode());
		buffer.WriteBytes(Function.Encode());

		return buffer;
	}
}
```

ModbusFrame 对应 ADU。Encode 方法用于在传输前对 ADU 编码。

开源地址：[modbus-tcp](https://github.com/VictorBu/modbus-tcp)