---
title: IDEA 远程调试
date: 2019-05-22 11:20:00
updated: 2019-05-22 11:20:00
categories: [IT]
tags: [IDE]
---
# 菜单 -> Run -> Edit Configurations...

![](https://oss.x8y.cc/blog-img/2072/1.png)

# 添加 -> Remote

![](https://oss.x8y.cc/blog-img/2072/2.png)

# 配置

![](https://oss.x8y.cc/blog-img/2072/3.png)

# 启动远程项目

正常启动命令如下：

```
java -jar ***.jar
```

开启远程调试：

```
java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=7015 **.jar
```

其中 -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=7015 是从上一步方框里拷贝出来的

# 注意事项

要保证远程调试监听的端口是放行状态


**本人 C# 转 Java 的 newbie, 如有错误或不足欢迎指正，谢谢**

