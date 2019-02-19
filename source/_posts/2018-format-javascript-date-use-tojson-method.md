---
title: JavaScript 使用 toJSON 方法格式化日期
date: 2019-02-19 21:00:00
updated: 2019-02-19 21:00:00
categories: [IT]
tags: [js]
---

toJSON 方法可以将 Date 对象转换为 ISO-8601 标准的字符串：YYYY-MM-DDTHH:mm:ss. sssZ

```
var date = new Date();
// toJSON() 返回的是 UTC 时间，所以需要提前修正
date.setMinutes(date.getMinutes() - date.getTimezoneOffset()); 
date.toJSON().substr(0, 19).replace(/[-T:]/g, ''); //YYYYMMDDHHmmss
```

+ getMinutes: 获取 Date 对象的分钟(0~59)
+ getTimezoneOffset: 获取本地时间与 UTC 时间的分钟差
+ setMinutes: date.getMinutes() - 90 表示设置 date 为 90 分钟之前的时间

> 参考：

1. [JS最简便日期格式化YYYYMMDD的方法](https://www.jianshu.com/p/c7fbe4e2676c)
