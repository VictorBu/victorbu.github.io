---
title: C# ��ȡ�ļ���ϸ��ע��Ϣ (��ͼƬ����Ƶʵ�ʴ���ʱ��)
date: 2019-02-03 10:12:00
updated: 2019-02-03 10:12:00
categories: [IT]
tags: [WindowsAPICodePack-Shell]
---

> ��������Ƭ/��Ƶʱ�����ʵ������ʱ���������ļ����� System.IO.FileInfo ֻ�ܻ�ȡ���ļ��Ĵ���ʱ������д��ʱ�䣬������Ҫ����Ѱ�ҽ������

# ���� 1: System.Drawing

1. [c#������������Ƭ����ȡ����ʱ��](https://blog.csdn.net/jiuzaizuotian2014/article/details/81487398)

```C#
static void Main(string[] args)
{
	var file = @"D:\image\IMG_6789.JPG";

	var image = Image.FromFile(file);
	var propItems = image.PropertyItems;

	var propItem = image.GetPropertyItem(0x9003); //Id Ϊ 0x9003 ��ʾ���յ�ʱ��
	var propItemValue = propItem.Value;

	image.Dispose();

	var dateTimeStr = System.Text.Encoding.ASCII.GetString(propItemValue).Trim('\0');

	var dt = DateTime.ParseExact(dateTimeStr, "yyyy:MM:dd HH:mm:ss", CultureInfo.InvariantCulture);

	Console.WriteLine(dt);
}
```

*�˷�����������ͼƬ��pass*

# ���� 2: shell32

1. [C#ͨ��shell32��ȡ�ļ���ϸ��ע��Ϣ](https://blog.csdn.net/u011127019/article/details/52169653)

1. [How to use Shell32 within a C# application?](How to use Shell32 within a C# application?)

1. [Exception when using Shell32 to get File extended properties](https://stackoverflow.com/questions/31403956/exception-when-using-shell32-to-get-file-extended-properties)

�������裺

1. ���shell32���� (C:\Windows\System32\shell32.dll �� �� VS ��������� .COM -> Microsoft Shell Controls and Automation)

1. ���� dll "Ƕ�뻥��������" Ϊ false

```C#
static void Main(string[] args)
{
	var file = @"D:\image\IMG_6789.JPG";

	var shell = new ShellClass();
	var dir = shell.NameSpace(Path.GetDirectoryName(file));
	var item = dir.ParseName(Path.GetFileName(file));

	var dateTimeStr = dir.GetDetailsOf(item, 12);// 12 Ϊ��Ƭ����ʱ��
}
```

*�˷���������ͼƬ����Ƶ(id ��ͬ)��������ʵ���л�ȡ����ʱ���ַ������������޷�ת����ʱ�䣺pass*

# ���� 3: Microsoft.WindowsAPICodePack.Shell (���÷���)

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

**�˷�����ȫ����Ҫ��֧���������͵��ļ���ͬʱҲ���Ի�ȡ�ļ���������Ϣ�������ߵ�**

**[��Դ��ַ](https://github.com/VictorBu/code-snippet/tree/master/csharp/file-taken-datetime)**
