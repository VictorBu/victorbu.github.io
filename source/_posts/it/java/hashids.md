---
title: Hashids java 版使用
date: 2022-01-26 10:00:00
updated: 2022-01-26 10:00:00
categories: [IT]
tags: [Hashids, Java]
---

在软件开发中 id 通常为 int 或者 long 类型，有时会有混淆 id 的需求，比如反爬虫。[Hashids](https://hashids.org) 是一个小型的开源库，可以将数字或者十六进制字符串转换成唯一的、非顺序的 id。

# 使用

## 添加依赖

```
<dependency>
  <groupId>org.hashids</groupId>
  <artifactId>hashids</artifactId>
  <version>1.0.3</version>
</dependency>
```

## 编码一个数字

```
Hashids hashids = new Hashids("this is my salt");
String hash = hashids.encode(12345L);
// 结果：NkK9
```

## 解码一个数字

```
Hashids hashids = new Hashids("this is my salt");
long[] numbers = hashids.decode("NkK9");
```

## 编码几个数字

```
Hashids hashids = new Hashids("this is my salt");
String hash = hashids.encode(683L, 94108L, 123L, 5L);
// 结果：aBMswoO2UB3Sj
```

## 指定编码结果的最小长度

```
Hashids hashids = new Hashids("this is my salt", 8);
String hash = hashids.encode(1L);
// 结果：gB0NV05e
```

## 指定编码结果使用的字母表

```
Hashids hashids = new Hashids("this is my salt", 0, "0123456789abcdef");
String hash = hashids.encode(1234567L);
// 结果：b332db5
```

## 编码十六进制字符串

```
Hashids hashids = new Hashids("This is my salt");
String hash = hashids.encodeHex("507f1f77bcf86cd799439011");
// 结果：goMYDnAezwurPKWKKxL2
```

## 解码十六进制字符串

```
Hashids hashids = new Hashids("This is my salt");
String objectId = hashids.decodeHex(hash);
```

# 注意事项

Java 版本是基于 JS 版本实现，因为 JS 对数字的范围限制是 2^53 - 1 (9007199254740991)，为了保持兼容，Java 版本也保留了此限制，如果大于此数字将抛出 IllegalArgumentException 异常。

如果想要编码大于 9007199254740991 的数字可以使用编码十六进制字符串的方法。