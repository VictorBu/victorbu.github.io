---
title: C# 获取文件详细备注信息 (如图片、视频实际创建时间)
date: 2019-02-03 10:12:00
updated: 2019-02-03 10:12:00
categories: [IT]
tags: [WindowsAPICodePack-Shell]
---

> 在整理照片/视频时想根据实际拍摄时间重命名文件，但 System.IO.FileInfo 只能获取到文件的创建时间或最后写入时间，不符合要求，遂寻找解决方案

# 方案 1: System.Drawing

1. [c#从相机拍摄的照片中提取拍摄时间](https://blog.csdn.net/jiuzaizuotian2014/article/details/81487398)

```C#
static void Main(string[] args)
{
	var file = @"D:\image\IMG_6789.JPG";

	var image = Image.FromFile(file);
	var propItems = image.PropertyItems;

	var propItem = image.GetPropertyItem(0x9003); //Id 为 0x9003 表示拍照的时间
	var propItemValue = propItem.Value;

	image.Dispose();

	var dateTimeStr = System.Text.Encoding.ASCII.GetString(propItemValue).Trim('\0');

	var dt = DateTime.ParseExact(dateTimeStr, "yyyy:MM:dd HH:mm:ss", CultureInfo.InvariantCulture);

	Console.WriteLine(dt);
}
```

*此方法仅适用于图片：pass*

# 方案 2: shell32

1. [C#通过shell32获取文件详细备注信息](https://blog.csdn.net/u011127019/article/details/52169653)

1. [How to use Shell32 within a C# application?](How to use Shell32 within a C# application?)

1. [Exception when using Shell32 to get File extended properties](https://stackoverflow.com/questions/31403956/exception-when-using-shell32-to-get-file-extended-properties)

操作步骤：

1. 添加shell32引用 (C:\Windows\System32\shell32.dll 或 在 VS 中添加引用 .COM -> Microsoft Shell Controls and Automation)

1. 设置 dll "嵌入互操作类型" 为 false

```C#
static void Main(string[] args)
{
	var file = @"D:\image\IMG_6789.JPG";

	var shell = new ShellClass();
	var dir = shell.NameSpace(Path.GetDirectoryName(file));
	var item = dir.ParseName(Path.GetFileName(file));

	var dateTimeStr = dir.GetDetailsOf(item, 12);// 12 为照片拍摄时间
}
```

*此方法适用于图片和视频(id 不同)，但是在实际中获取到的时间字符串包含乱码无法转换成时间：pass*

# 方案 3: Microsoft.WindowsAPICodePack.Shell (采用方案)

1. [how-do-i-find-the-date-a-video-avi-mp4-was-actually-recorded](https://stackoverflow.com/questions/3104641/how-do-i-find-the-date-a-video-avi-mp4-was-actually-recorded)

1. [Microsoft.WindowsAPICodePack-Shell](https://www.nuget.org/packages/Microsoft.WindowsAPICodePack-Shell/)

```C#
static void Main(string[] args)
{
	var file = @"D:\image\IMG_6789.JPG";

	ShellObject obj = ShellObject.FromParsingName(file);
	var takenDate = obj.Properties.System.ItemDate.Value;

	Console.WriteLine(takenDate);

	Console.ReadLine();
}
```

**此方法完全符合要求，支持所有类型的文件。同时也可以获取文件的其他信息，如作者等**

**[开源地址](https://github.com/VictorBu/code-snippet/tree/master/csharp/file-taken-datetime)**
