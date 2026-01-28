---
title: Windows 访问 CentOS 7 共享文件夹 Samba 配置
date: 2018-02-27 00:00:00
updated: 2017-02-27 00:00:00
categories: [IT]
tags: [Samba, CentOS 7, Linux, Windows]
---

> Windows 使用用户名、密码访问 CentOS 7 共享文件夹

1. 执行命令，查看 Windows 工作组：`net config workstation`
1. 执行命令，安装 Samba：`yum install samba samba-client samba-common`
1. 执行命令，允许 Samba 穿透防火墙：

    `firewall-cmd --permanent --zone=public --add-service=samba`

    `firewall-cmd --reload`

1. 执行命令，新增用户：`useradd shareuser`
1. 执行命令，新增用户组：`groupadd smbgrp`
1. 执行命令，将用户加入用户组：`usermod shareuser -aG smbgrp`
1. 执行命令，设置用户访问共享文件夹的密码：`smbpasswd -a shareuser`
1. 执行命令，新建共享文件夹：`mkdir -p /srv/samba/secure`
1. 执行命令，修改文件夹权限：`chmod -R 0770 /srv/samba/secure`
1. 执行命令，改变文件夹所属组：`chown -R root:smbgrp /srv/samba/secure`
1. 执行命令，修改文件夹的安全上下文：`chcon -t samba_share_t /srv/samba/secure`
1. 执行命令，备份配置文件：`cp /etc/samba/smb.conf /etc/samba/smb.conf.orig`
1. 执行命令，修改配置文件：`vi /etc/samba/smb.conf`：

    + 修改 workgroup 为 Windows 的工作组：

        ```
        workgroup = 你的工作组名称
        ```
    
    + 添加或修改下列配置：
        ```
        [Secure]
        comment = Secure File Server Share
        path =  /srv/samba/secure
        valid users = @smbgrp
        guest ok = no
        writable = yes
        browsable = yes
        ```

1. 执行命令，验证配置是否正确：`testparm`
1. 执行命令，启动服务：

    `systemctl restart smb.service`
    `systemctl restart nmb.service`

1. 执行命令，设置服务开机启动：

    `chkconfig smb on`
    `chkconfig nmb on`


参考
----

1. [How to Install Samba4 on CentOS 7 for File Sharing on Windows](https://www.tecmint.com/install-samba4-on-centos-7-for-file-sharing-on-windows)
1. [Samba Server Installation and Configuration on CentOS 7](https://www.howtoforge.com/samba-server-installation-and-configuration-on-centos-7)