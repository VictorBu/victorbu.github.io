---
title: Netty 中 LengthFieldBasedFrameDecoder 构造函数取值备忘
date: 2019-04-28 17:00:00
updated: 2019-04-28 17:00:00
categories: [IT]
tags: [Java, Netty]
---

```
public LengthFieldBasedFrameDecoder(ByteOrder byteOrder, 
    int maxFrameLength, int lengthFieldOffset, int lengthFieldLength, int lengthAdjustment, 
    int initialBytesToStrip, boolean failFast) {
    ... ...
}
```

+ byteOrder: Length 字段是大端还是小端，默认是 ByteOrder.BIG_ENDIAN
+ maxFrameLength: 完整数据包的最大长度
+ lengthFieldOffset: Length 字段起始索引
+ lengthFieldLength: Length 字段长度
+ lengthAdjustment: 默认情况下 Length 的长度表示的是 Length 字段之后的长度，如果 Length 长度包含的是真个数据包的长度或没有包含结束符等，则需要调整该值
+ initialBytesToStrip: 识别出整个数据包后，跳过的长度 (通常用来跳过数据包包头，只解析正文)
+ failFast: 一般设置为 true，读到 Length 后，如果 Length 超过 maxFrameLength，则立即抛出异常


# 举例

```
+--------------+--------------+------------+------------+----------+---------------+----------+
| ResponseCode | FunctionCode | ParamType  | StatusCode | Length   | Data          | EOF      |
| (2 Byte)     | (2 Byte)     | (1 Byte)   | (1 Byte)   | (2 Byte) | (Length Byte) | (2 Byte) |
+--------------+--------------+------------+------------+----------+---------------+----------+
```

+ lengthFieldOffset = 6 (2 + 2 + 1 + 1)
+ lengthFieldLength = 2 (Length 字段本身长度)
+ lengthAdjustment = 2 (EOF 长度：本例中 Length 字段表示的长度仅表示了 Data 部分，没有包含 EOF)

参考：[【看完就会】Netty的LengthFieldBasedFrameDecoder的用法详解](https://blog.csdn.net/zougen/article/details/79037675)



