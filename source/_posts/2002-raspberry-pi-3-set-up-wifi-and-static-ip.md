---
title: 树莓派 3 设置 WIFI 及静态 IP
date: 2018-02-03 00:00:00
updated: 2017-02-03 00:00:00
categories: [IT]
tags: [树莓派]
---


# 设置 WIFI

1. 执行命令：`sudo vi /etc/wpa_supplicant/wpa_supplicant.conf`

    在文件底部添加：

    ```
    network={
        ssid="WIFI 名称"
        psk="WIFI 密码"
    }
    ```


# 设置静态 IP

1. 执行命令，查看路由信息，记下网关配置：`route -ne`

    ![image](https://oss.x8y.cc/blog-img/2002/route.png)

1. 执行命令，查看域名服务器列表，记下列出的域名服务器：`cat /etc/resolv.conf`

    ![image](https://oss.x8y.cc/blog-img/2002/resolv.png)

1. 执行命令：`sudo vi /etc/dhcpcd.conf`

    在文件底部添加：

    ```
    interface wlan0
    static ip_address=192.168.1.66
    static routers=192.168.1.1
    static domain_name_servers=192.168.1.1
    ```

    ![image](https://oss.x8y.cc/blog-img/2002/dhcpcd.png)

1. 执行命令，重启设备：`sudo reboot`

1. 直接登录或者使用配置的 ip 重新连接，执行命令，确认配置是否正确：`ping github.com`


参考
----

1. [SETTING WIFI UP VIA THE COMMAND LINE](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)
1. [HOW TO SET UP A STATIC IP ON THE RASPBERRY PI](http://www.circuitbasics.com/how-to-set-up-a-static-ip-on-the-raspberry-pi)
