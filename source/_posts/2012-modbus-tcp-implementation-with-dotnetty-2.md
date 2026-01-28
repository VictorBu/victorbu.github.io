---
title: DotNetty 实现 Modbus TCP 系列 (二) ModbusFunction 类图及继承举例
date: 2019-02-13 16:00:00
updated: 2019-02-13 16:00:00
categories: [IT]
tags: [DotNetty, Modbus]
---

ModbusFunction 类图如下：

![](https://oss.x8y.cc/blog-img/2012/modbusfunction.png)

如前文所述，所有请求/相应的 PDU 均继承自 ModbusFunction，其子类传入对应的 Function Code 并实现三个方法：

1. CalculateLength：Data 部分的长度(该方法也可以为属性，但属性没有强制性，怕漏掉故改为抽象方法)
1. Decode：从缓冲区解析 Data
1. Encode：在传输前对 Data 编码

# 实现举例

每个 Function Code 均对应 ModbusFunction 的两个子类：请求类和响应类，以 0x03(读取保持寄存器值)为例：

## 请求类

请求报文 Data 说明：

![](https://oss.x8y.cc/blog-img/2012/request.png)


```C#
public class ReadHoldingRegistersRequest : ModbusFunction
{
	public ushort StartingAddress { get; private set; }
	public ushort Quantity { get; private set; }

	public ReadHoldingRegistersRequest()
		: base((short)ModbusCommand.ReadHoldingRegisters)
	{

	}

	public ReadHoldingRegistersRequest(ushort startingAddress, ushort quantity)
		: base((short)ModbusCommand.ReadHoldingRegisters)
	{
		StartingAddress = startingAddress;
		Quantity = quantity;
	}

	public override int CalculateLength()
	{
		return 2 + 2; // StartingAddress Length + Quantity Length
	}
	public override void Decode(IByteBuffer buffer)
	{
		StartingAddress = buffer.ReadUnsignedShort();
		Quantity = buffer.ReadUnsignedShort();
	}
	public override IByteBuffer Encode()
	{
		IByteBuffer buffer = Unpooled.Buffer();
		buffer.WriteByte(FunctionCode);

		buffer.WriteUnsignedShort(StartingAddress);
		buffer.WriteUnsignedShort(Quantity);

		return buffer;
	}
}
```

## 响应类

响应报文 Data 说明：

![](https://oss.x8y.cc/blog-img/2012/response.png)

```C#
public class ReadHoldingRegistersResponse : ModbusFunction
{
	private ushort byteCount;
	public ushort[] Registers { get; private set; }

	public ReadHoldingRegistersResponse()
		: base((short)ModbusCommand.ReadHoldingRegisters)
	{

	}

	public ReadHoldingRegistersResponse(ushort[] registers)
		: base((short)ModbusCommand.ReadHoldingRegisters)
	{
		Registers = registers;
		byteCount = (ushort)(registers.Length * 2);
	}

	public override int CalculateLength()
	{
		return 1 + byteCount;
	}

	public override void Decode(IByteBuffer buffer)
	{
		byteCount = buffer.ReadByte();
		Registers = new ushort[byteCount / 2];
		for (int i = 0; i < Registers.Length; i++)
		{
			Registers[i] = buffer.ReadUnsignedShort();
		}

	}

	public override IByteBuffer Encode()
	{
		IByteBuffer buffer = Unpooled.Buffer();
		buffer.WriteByte(FunctionCode);
		buffer.WriteByte(byteCount);

		foreach (var register in Registers)
		{
			buffer.WriteUnsignedShort(register);
		}

		return buffer;
	}
}
```

其中 ModbusCommand 为 Function Code 的枚举：

```C#
enum ModbusCommand : short
{

	ReadCoils = 0x01,
	ReadDiscreteInputs = 0x02,
	ReadHoldingRegisters = 0x03,
	ReadInputRegisters = 0x04,
	WriteSingleCoil = 0x05,
	WriteSingleRegister = 0x06,

	WriteMultipleCoils = 0x0F,
	WriteMultipleRegisters = 0x10,

}
```

文中为方便理解请求类和响应类均直接继承 ModbusFunction，实际开发中请求类和响应类均没有直接继承 ModbusFunction，而是根据其他 Function Code 的 Data 进行再次抽象后继承。

开源地址：[modbus-tcp](https://github.com/VictorBu/modbus-tcp)