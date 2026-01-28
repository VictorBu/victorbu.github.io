---
title: Hyper-V 中设置虚拟机静态 IP
date: 2020-05-13 11:00:00
updated: 2020-05-13 11:00:00
categories: [IT]
tags: [Hyper-V]
---

# 一、新建虚拟网络交换机

![](https://oss.x8y.cc/blog-img/2107/adaper.png)

# 二、配置网络

![](https://oss.x8y.cc/blog-img/2107/network.png)

网络共享默认使用 192.168.137.0/255 作为内网地址，192.168.137.1 作为网关

![](https://oss.x8y.cc/blog-img/2107/ip.png)


# 三、配置虚拟机静态 IP

安装完成虚拟机后修改配置文件：

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static" # 静态 ip
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="eth0"
UUID="b4857ed1-72c8-4845-871d-c94248cf9a3a"
DEVICE="eth0"
ONBOOT="yes" # 开机启动
IPADDR="192.168.137.101" # 本机 ip
NETMASK="255.255.255.0" # 子网掩码
GATEWAY="192.168.137.1" # 网关
DNS1="192.168.137.1" # DNS
```

有注释的是需要添加或修改的配置