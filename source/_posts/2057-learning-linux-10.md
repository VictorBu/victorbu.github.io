---
title: Linux 学习 (十) 网络配置
date: 2019-03-26 16:00:00
updated: 2019-03-26 16:00:00
categories: [IT]
tags: [Linux]
---

> [Linux网络管理](https://www.imooc.com/learn/258) 学习笔记



# 配置 IP 地址

## ifconfig 命令临时配置 IP 地址

ifconfig eth0 192.168.0.200 netmask 255.255.255.0 #临时设置 eth0 网卡的 IP 地址与子网掩码

## setup 工具永久配置 IP 地址

## 修改网络配置文件

1. 网卡信息文件：/etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0 #网卡设备名

BOOTPROTO=none #是否自动获取 IP(none, static, dhcp)

HWADDR=00:0c:29:17:c4:09 #MAC 地址

NM_CONTROLLED=yes #是否可以由 Network Manager 图形管理工具托管

ONBOOT=yes #是否随网络服务启动，eth0 生效

TYPE=Ethernet #类型为以太网

UUID="..." #唯一识别码

IPADDR=192.168.0.252 # IP 地址

NETMASK=255.255.255.0 #子网掩码

GATEWAY=192.168.0.1 #网管

DNS1=202.106.0.20 #DNS

IPV5INT=no #IPv6 没有启用

USERCTL=no #不允许非 root 用户控制此网卡

2. 主机名文件：/etc/sysconfig/network

NETWORKING=yes

HOSTNAME=localhost.localdomain

**hostname [主机名] ：查看与临时设置主机名**

3. DNS 配置文件：/etc/resolv.conf

nameserver 202.106.0.20

search localhost

## 图形界面配置 IP 地址


# 虚拟机网络配置

1. 配置 IP 地址
2. 启动网卡
    1. ONBOOT=yes
    2. 重启网络服务：service network restat
3. 修改 UUID
    1. /etc/sysconfig/network-scripts/ifcfg-eth0 #删除 MAC 地址行
    2. rm -rf /etc/udev/rules.d/70-persistent-net.rules #删除网卡和 MAC 地址绑定文件
    3. 重启系统
4. 设置虚拟机网络连接方式
    1. 桥接
    2. NAT
    3. Host-only
5. 修改桥接网卡