---
title: 在 Windows 中使用 C# 启动其他程序
date: 2019-05-13 17:00:00
updated: 2019-05-13 17:00:00
categories: [IT]
tags: [Shell32]
---

> 因为某些原因需要自动启动一个 Winform 程序，可能是因为第三方资源的原因，使用 System.Diagnostics.Process 无法成功启动 (可以看到界面，但是会报  Unhandled Exception)

# 解决方案 (使用 Shell32)

导入方法

```
[DllImport("shell32.dll")]
public static extern int ShellExecute(IntPtr hwnd, StringBuilder lpszOp, StringBuilder lpszFile, StringBuilder lpszParams, StringBuilder lpszDir, int FsShowCmd);
```

调用

```
ShellExecute(IntPtr.Zero, new StringBuilder("Open"), new StringBuilder("可执行文件名"), new StringBuilder(""), new StringBuilder("文件所在目录"), 1);
```

参考

[C# winform 启动外部程序](http://www.cnblogs.com/zhujiantao/p/6694446.html)


