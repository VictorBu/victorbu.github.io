---
title: Linux 学习 (五) 压缩与解压缩命令
date: 2019-03-26 11:00:00
updated: 2019-03-26 11:00:00
categories: [IT]
tags: [Linux]
---

> [Linux达人养成计划 I](https://www.imooc.com/learn/175) 学习笔记



常用压缩格式：.zip | .gz | .bz2 | .tar.gz | .tar.bz2

# .zip

+ zip 压缩文件名 源文件：压缩文件
+ zip -r 压缩文件名 源目录：压缩目录
+ upzip 压缩文件：解压缩

# .gz

+ gzip 源文件：源文件会消失
+ gzip -c 源文件 > 压缩文件：源文件保留
+ gzip -r 目录：压缩目录下所有的子文件，但是不能压缩目录
+ gzip -d 压缩文件：解压缩文件
+ gunzip 压缩文件：解压缩文件

# .bz2

+ bzip2 源文件：不保留源文件
+ bzip2 -k 源文件：保留源文件
+ bzip2 -d 压缩文件：解压缩，-k 保留压缩文件
+ bunzip2 压缩文件：解压缩，-k 保留压缩文件

注：bzip2命令不能压缩目录

# .tar.gz / .tar.bz2

## tar -cvf 打包文件名 源文件

+ -c：打包
+ -v：显示过程
+ -f：指定打包后的文件名

打包后再压缩为.gz或.bz2

## tar -xvf 打包文件名

+ -x：解打包

## tar -zcvf 压缩包名.tar.gz 源文件

+ -z：压缩为.tar.gz格式

## tar -zxvf 压缩包名.tar.gz

+ -x：解压缩

## tar -jcvf 压缩包名.tar.bz2 源文件

+ -j：压缩为.tar.bz2格式

## tar -jxvf 压缩包名.tar.bz2

+ -x：解压缩

tar 可以同时压缩多个文件，也可以解压到指定目录

## tar -ztvf 压缩包名.tar.gz / tar -jtvf 压缩包名.tar.gz

+ 只查看压缩包内容，不解压
