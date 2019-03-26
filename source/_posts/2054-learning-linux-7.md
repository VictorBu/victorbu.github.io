---
title: Linux 学习 (七) 挂载命令 & 用户登陆查看
date: 2019-03-26 13:00:00
updated: 2019-03-26 13:00:00
categories: [IT]
tags: [Linux]
---

> [Linux达人养成计划 I](https://www.imooc.com/learn/175) 学习笔记



# 挂载命令

## mount：查询系统中已经挂载的设备

## mount -a：根据配置文件 /etc/fstab 的内容，自动挂载

## mount [-t 文件系统] [-o 特殊选项] 设备文件名 挂载点

+ -t 文件系统：加入文件系统类型来指定挂载的类型，可以是ext3、ext4、iso9660等文件系统
+ -o 特殊选项：可以指定挂载的额外选项


参数 | 说明
---|---
atime/noatime | 更新访问时间/不更新访问时间。访问分区文件时，是否更新文件的访问时间，默认为更新
async/sync | 异步/同步，默认为异步
auto/noauto | 自动/手动，mount -a 命令执行时，是否会自动安装 /etc/fstab 文件内容挂载，默认为自动
defaults | 定义默认值，相当于 rw, suid, dev, exec, auto, nouser, async 这七个选项
exec/noexec | 执行/不执行，设定是否允许在文件系统中执行可执行文件，默认是 exec 允许
remount | 重新挂载已经挂载的文件系统，一般用于指定修改特殊权限
rw/ro | 读写/只读，文件系统挂载时，是否具有读写权限，默认是rw
suid/nosuid | 具有/不就有 SUID 权限，设定文件系统是否具有SUID 和 SGID 的权限，默认是具有
user/nouser | 允许/不允许普通用户挂载，设定文件系统是否允许普通用户挂载，默认是不允许，只有root可以挂载分区
usrquota | 写入代表文件系统支持用户磁盘配额，默认不支持
grpquota | 写入代表文件系统支持组磁盘配额，默认不支持


挂载光盘：

+ mkdir /mnt/cdrom/ ：建立挂载点
+ mount -t iso9660 /dev/cdrom /mnt/cdrom/ ：挂载光盘
注：上面一条命令也可以写为：mount /dev/sr0 /mnt/cdrom/

挂载U盘：

+ fdisk -l ：查看U盘设备文件名
+ mount -t vfat /dev/sdb1 /mnt/usb/ ：不一定是sdb1
注：Linux 默认是不支持 NTFS 文件系统的

## umount 设备文件名或挂载点：卸载命令


# 用户登陆查看

## w 用户名：查看登陆用户信息

+ USER：登陆的用户名
+ TTY：登陆终端
+ FROM：从哪个IP地址登陆
+ LOGIN@：登陆时间
+ IDLE：用户闲置时间
+ JCPU：指的是和该终端所连接的所有进程占用的时间。这个时间里并不包括过去的后台作业时间，但却包括当前正在运行的后台作业所占用的时间
+ PCPU：是指当前进程所占用的时间
+ WHAT：当前正在运行的命令

## who 用户名：查看登陆用户信息

+ 用户名
+ 登陆终端
+ 登陆时间(登陆来源IP地址)

## last：查询当前登陆和过去登陆的用户信息

+ 默认是读取 /var/log/wtmp 文件数据
+ 用户名
+ 登陆终端
+ 登陆IP
+ 登陆时间
+ 退出时间(在线时间)

## lastlog：查询所有用户的最后一次登陆时间

+ 默认是读取 /var/log/lastlog 文件数据
+ 用户名
+ 登陆终端
+ 登陆IP
+ 最后一次登陆时间

