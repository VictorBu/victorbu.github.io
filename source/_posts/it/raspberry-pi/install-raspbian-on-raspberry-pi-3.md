---
title: 树莓派 3 安装 Raspbian
date: 2018-02-01 00:00:00
updated: 2017-02-01 00:00:00
categories: [IT]
tags: [树莓派, Raspbian]
---


# 前期准备

1. [SD Card Formatter](https://www.sdcard.org/downloads/formatter_4/eula_windows/index.html)

1. [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager)

1. [Raspbian](https://www.raspberrypi.org/downloads/raspbian)


# 开始安装

1. 使用 SD Card Formatter 格式化 Micro SD 卡

    ![image](https://oss.x8y.cc/blog-img/2001/SD_Card_Formatter.png)

1. 解压 raspbian.zip ，得到 img格式的镜像文件

1. 写入系统

    ![image](https://oss.x8y.cc/blog-img/2001/Win32_Disk_Imager.png)

1. 打开Micro SD 卡，在根目录新建名为 "ssh" 的文件，开启 ssh 服务


# 登录/远程连接

默认用户名：pi

默认密码：raspberry 


# 设置时区 (可选)

1. 执行命令：`date`

    如果输出时间与当前时间不一致则需要修改

1. 执行命令：`sudo dpkg-reconfigure tzdata`

    调出配置界面，选择配置(Asia, Shanghai)：

    ![image](https://oss.x8y.cc/blog-img/2001/Configuring_tzdata_Geographical_area.png)

    ![image](https://oss.x8y.cc/blog-img/2001/Configuring_tzdata_Time_zone.png)


# 启用 root 远程登录 (可选)

1. 执行命令：`sudo passwd root`

1. 输入两遍密码，然后执行命令：`sudo passwd --unlock root`


# 允许 root 远程 (可选)

1. 执行命令：`sudo vi /etc/ssh/sshd_config`

    添加或编辑：

    ```
    PermitRootLogin yes
    ```

1. 保持退出后，执行命令，重启 SSH：`service ssh restart`


# 更换软件源 (可选)

1. 执行命令：`sudo vi /etc/apt/sources.list`

    切换为阿里云的软件源：

    http://mirrors.aliyun.com/raspbian/raspbian/

    修改前：

    ![image](https://oss.x8y.cc/blog-img/2001/apt_sources_list_before.png)

    修改后：

    ![image](https://oss.x8y.cc/blog-img/2001/apt_sources_list_after.png)

   添加的两行：

    ```
    deb http://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi
    dep-src http://mirrors.aliyun.com/raspbian/raspbian/ stretch main contrib non-free rpi
   ```

1. 执行下面三个命令更新源、已安装的包和系统：

    `sudo apt-get -y update`

    `sudo apt-get -y upgrade`

   `sudo apt-get -y dist-upgrade`

# 开启 ftp (可选)

1. 执行命令，安装 vsftpd 服务器：`sudo apt-get install vsftpd`
1. 执行命令，启动 ftp 服务：`sudo service vsftpd start`
1. 执行命令：`sudo vi /etc/vsftpd.conf`

    编辑：
    ```
    anonymous_enable=NO
    local_enable=YES
    write_enable=YES
    local_umask=022
    ```
1. 执行命令，重启 vsftpd 服务：`sudo service vsftpd restart`

参考
----

1. [树莓派入门—安装系统](https://jingyan.baidu.com/article/ca00d56c384ca0e99febcf43.html/)
1. [win10下用xshell初次连接树莓派](http://blog.csdn.net/u012837829/article/details/53946224)
1. [ 设置树莓派时区（中国时区）](http://blog.csdn.net/faryang/article/details/50779346)
1. [玩转树莓派01——初始化](https://www.jianshu.com/p/2f80d4003c1d)
1. [树莓派进阶之路 (008) - 树莓派安装ftp服务器(转)](https://www.cnblogs.com/jikexianfeng/p/5862130.html)
