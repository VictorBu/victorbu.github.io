---
title: Linux 学习 (六) 关机与重启命令
date: 2019-03-26 12:00:00
updated: 2019-03-26 12:00:00
categories: [IT]
tags: [Linux]
---

> [Linux达人养成计划 I](https://www.imooc.com/learn/175) 学习笔记


# shutdown [选项] 时间

+ -c：取消前一个关机命令
+ -h：关机
+ -r：重启

shutdown命令会在关机或重启时自动保存系统中正在运行的服务，最安全的关机或重启命令。

# half：关机

# poweroff：关机

# init 0：关机

# reboot：重启(相对安全)

# init 6：重启

系统运行级别：

+ 0：关机
+ 1：单用户
+ 2：不完全多用户，不含NFS服务
+ 3：完全多用户
+ 4：未分配
+ 5：图形界面
+ 6：重启

# runlevel：查看系统运行级别

# logout：退出登陆