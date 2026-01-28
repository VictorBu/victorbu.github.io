---
title: Linux 学习 (一) Linux简介
date: 2019-03-26 07:00:00
updated: 2019-03-26 07:00:00
categories: [IT]
tags: [Linux]
---

> [Linux达人养成计划 I](https://www.imooc.com/learn/175) 学习笔记


Linux 内核官网：[www.kernel.org](https://www.kernel.org)

内核版本说明：主版本.次版本.末版本，如2.6.18

# Linux 主要发行版本

+ RedHat: 服务器领域，部分功能收费
+ Ubuntu: 图形界面
+ CentOS: 与RedHat功能一样，免费
+ Fedora: 个人版本，功能比RedHat更强大

Netcraft: [www.netcraft.com](https://www.netcraft.com)

# Linux 与 Windows的不同

+ Linux 严格区分大小写
+ Linux 中所有内容(包括硬件)以文件形式保存，即一切内容皆文件
+ Linux 不以扩展名区分文件类型

# 约定俗成的扩展名

+ 压缩包: "*.gz", "*.bz2", "*.tar.bz2", ".tgz"
+ 二进制软件包: ".rpm"
+ 脚本文件: "*.sh"
+ 配置文件: "*.conf"

# 分区类型

+ 主分区: 最多只能有4个
+ 扩展分区: 
    + 最多只能有1个
    + 主分区加扩展分区最多有4个
    + 不能写入数据，只能包含逻辑分区
+ 逻辑分区

# 硬件设备文件名


硬件 | 硬件设备名
---|---
IDE硬盘 | /dev/hd[a-d]
SCSI/SATA/USB硬盘 | /dev/sd[a-p]
光驱 | /dev/cdrom或/dev/hdc
软盘 | dev/fd[0-1]
打印机(25针) | dev/lp[0-2]
打印机(USB) | dev/usb/lp[0-15]
鼠标 | /dev/mouse

分区设备文件名：

+ /dev/hda1 (IDE硬盘接口)
+ /dev/sda1 (SCSI硬盘接口/SATA硬盘接口)
    
注：a代表第一块硬盘，1代表第一个分区

# 挂载：给每个分区分配挂载点

+ 必须分区
    + / (根分区)
    + swap分区 (交换分区，4G以内为内存2倍，4G以上同内存大小)
+ 推荐分区
    + /boot (启动分区，200MB)

# 安装日志

+ /root/install.log: 存储了安装在系统中的软件包及其版本信息
+ /root/install.log.syslog: 存储了安装过程中留下的事件记录
+ /root/anaconda-ls.cfg: 以Kickstart配置文件的格式记录安装过程中设置的选项信息

# 命令格式

命令 [选项] [参数]

注意：

+ 个别命令使用不遵循此格式
+ 当有多个选项时，可以写在一起
+ 简化选项与完整选项，例如 -a 等于 --all

