---
title: Modbus CRC 16 (C#)
date: 2019-02-17 22:30:00
updated: 2019-02-17 22:30:00
categories: [IT]
tags: [Modbus, CRC]
---

# 算法

1．预置一个值为 0xFFFF 的 16 位寄存器，此寄存器为 CRC 寄存器。
2．把第 1 个 8 位二进制数据（即通信消息帧的第 1 个字节）与 16 位的 CRC 寄存器相异或，异或的结果仍存放在该 CRC 寄存器中。
3．把 CRC 寄存器的内容右移一位，用 0 填补最高位，并检测移出位是 0 还是 1.
4．如果移出位为0 ，则重复步骤（3）（再次右移一位）；如果移出位为 1，则 CRC 寄存器与 0xA001 （多项式码）进行异或。
5．重复步骤（3）和（4），直到右移 8 次，这样整个 8 位数据全部进行了处理。
6．重复步骤（2）~（5），进行消息帧下一个字节的处理。
7．将该通信消息帧所有字节按上述步骤计算完成后，得到的 16 位 CRC 寄存器的高、低位进行交换。即发送时首先添加低位字节，然后添加高位字节。
8．最后得到的 CRC 寄存器内容即为CRC 校验码。

# 代码

```C#
public static byte[] GetModbusCrc16(byte[] bytes)
{
	byte crcRegister_H = 0xFF, crcRegister_L = 0xFF;// 预置一个值为 0xFFFF 的 16 位寄存器

	byte polynomialCode_H = 0xA0, polynomialCode_L = 0x01;// 多项式码 0xA001

	for (int i = 0; i < bytes.Length; i++)
	{
		crcRegister_L = (byte)(crcRegister_L ^ bytes[i]);

		for (int j = 0; j < 8; j++)
		{
			byte tempCRC_H = crcRegister_H;
			byte tempCRC_L = crcRegister_L;

			crcRegister_H = (byte)(crcRegister_H >> 1);
			crcRegister_L = (byte)(crcRegister_L >> 1);
			// 高位右移前最后 1 位应该是低位右移后的第 1 位：如果高位最后一位为 1 则低位右移后前面补 1
			if ((tempCRC_H & 0x01) == 0x01)
			{
				crcRegister_L = (byte)(crcRegister_L | 0x80);
			}

			if ((tempCRC_L & 0x01) == 0x01)
			{
				crcRegister_H = (byte)(crcRegister_H ^ polynomialCode_H);
				crcRegister_L = (byte)(crcRegister_L ^ polynomialCode_L);
			}
		}
	}

	return new byte[] { crcRegister_L, crcRegister_H };
}
```

代码地址：[ModbusCrc16](https://github.com/VictorBu/code-snippet/tree/master/csharp/ModbusCrc16)