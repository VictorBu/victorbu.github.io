---
title: Linux 学习 (十一) 软件安装管理
date: 2019-03-26 17:00:00
updated: 2019-03-26 17:00:00
categories: [IT]
tags: [Linux]
---

> [Linux软件安装管理](https://www.imooc.com/learn/447) 学习笔记




# 软件包简介

软件包分类：

+ 源码包 ：脚本安装包
+ 二进制包(RPM 包、系统默认包)

源码包的优点：

- 开源，如果有足够的能力，可以修改源代码
- 可以自由选择所需的功能
- 软件是编译安装，所以更适合自己的系统，更加稳定，也效率更高
- 卸载方便

源码包的缺点：

- 安装过程步骤较多，尤其安装较大的软件集合时(如 LAMP)，容易出现拼写错误
- 便宜过程时间较长，安装比二进制安装时间长
- 因为是编译安装，安装过程中一旦报错新手很难解决


二进制包的优点：

- 包管理系统简单，只通过几个命令就可以实现包的安装、升级、查询和卸载
- 安装速度比源码包安装快的多

二进制包的缺点：

- 经过编译，不再可以看到源代码
- 功能选择不如源代码灵活
- 依赖性

脚本安装包：把复杂的软件包安装过程写成了程序脚本，初学者可以执行程序脚本实现一键安装。但实际安装的还是源码包和二进制包。

- 优点：安装简单、快捷
- 缺点：完全丧失了自定义性


# rpm 命令管理

rpm 包的来源 ：在系统光盘中(/mnt/cdrom/Packages)

rpm 包命名规则 ：httpd-2.2.15-15.el6.centos.1.i686.rpm

+ httpd ：软件包名
+ 2.2.15 ：软件版本
+ 15 ：软件发布的次数
+ el6.centos ：适合的 Linux 平台
+ i686 ：适合的硬件平台
+ rpm ：包扩展名

rpm 包依赖性：

+ 树形依赖 ：a->b->c
+ 环形依赖 ：a->b->c->a
+ 模块依赖 ：模块依赖查询网站 [www.rpmfind.net](http://www.rpmfind.net)

包全名与包名：

+ 包全名：操作的包是没有安装的软件包时，使用包全名。而且要注意路径
+ 包名：操作已经安装的软件包时，使用包名，是搜索 /var/lib/rpm/ 中的数据库

## 安装

**rpm -ivh 包全名**
+ -i(install)：安装
+ -v(verbose)：显示详细信息
+ -h(hash)：显示进度
+ --nodeps：不检测依赖性

## 升级

**rmp -Uvh 包全名**
+ -U(upgrade) ：升级

## 卸载

**rmp -e 包名**
+ -e(erase) ：卸载
+ --nodeps ：不检查依赖性

## rpm 包查询

**rpm -q 包名 ：查询包是否安装**
+ -q(query) ：查询

**rpm -qa ：查询所有已经安装的 rpm 包**
+ -a(all) ：所有

**rpm -qi 包名 ：查询软件包详细信息**
+ -i(information) ：查询软件信息
+ -p(package) ：查询未安装包信息

**rpm -ql 包名 ：查询软件包安装位置**
+ -l(list) ：列表
+ -p(package) ：查询未安装包信息

rpm 包默认安装路径：
路径 | 释义
---|---
/etc/ | 配置文件安装目录
/usr/bin/ | 可执行的命令安装目录
/usr/lib/ | 程序所使用的函数库保存位置
/usr/share/doc/ | 基本的软件使用手册保存位置
/usr/share/man/ | 帮助文件保存位置

**rpm -qf 系统文件名 ：查询系统文件属于哪个 rpm 包**
+ -f(file) ：查询系统文件属于哪个软件包

**rpm -qR 包名 ：查询软件包的依赖性**
+ -R(requires) ：查询软件包的依赖性
+ -p(package) ：查询未安装包信息

## rpm 包校验

**rpm -V 已安装的包名**

+ -V(verify) ：校验指定 rpm 包中的文件

验证内容的8个信息：

+ S ：文件大小是否改变
+ M ：文件的类型或文件的权限(rwx)是否被改变
+ 5 ：文件 MD5 校验是否改变(可以看成文件内容是否改变)
+ D ：设备的主从代码是否改变
+ L ：文件路径是否改变
+ U ：文件的属主(所有者)是否改变
+ G ：文件的属组是否改变
+ T ：文件的修改时间是否改变

文件类型：

+ c ：配置文件(config file)
+ d ：普通文档(documentation)
+ g ：ghost file，很少见，就是该文件不应该被这个 rpm 包含
+ L ：授权文件(license file)
+ r ：描述文件(read me)

**rpm2cpio 包全名 | cpio -idv .文件绝对路径 ：rpm 包中文件提取**
+ rpm2cpio ：将 rpm 包转换成 cpio 格式的命令
+ cpio ：是一个标准工具，它用于创建软件档案文件和从档案文件中提取文件

**cpio 选项 < [文件|设备]**
+ -i ：copy-in 模式，还原
+ -d ：还原时自动新建目录
+ -v ：显示还原过程


# yum 在线安装

将所有软件包放到官方服务器上，当进行 yum 在线安装时，可以自动解决依赖性问题

redhat 的 yum 在线安装需要付费

## yum 源文件

**/etc/yum.repos.d/CentOS-Base.repo**
+ [base] ：容器名称，一定要放在[]中
+ name ：容器说明，可以自己随便写
+ mirrorlist ：镜像站点，这个可以注释掉
+ baseurl ：yum 源服务器地址。默认是 CentOS 官方的 yum 源服务器，是可以使用的，如果觉得慢可以改成其他 yum 源地址
+ enabled ：此容器是否生效，如果不写或写成 enabled = 1 都是生效，写成 enabled = 0 就是不生效
+ gpgcheck ：如果 1 是指 RPM 的数字证书生效，如果是 0 则不生效
+ gpgkey ：数字证书的公钥文件保存位置，不用修改

## 光盘搭建 yum 源

1. 挂载光盘
2. 使网络 yum 源失效
    + cd /etc/yum.repos.d/ #进入 yum源目录
    + mv CentOS-Base.repo CentOS-Base.repo.bak #修改 yum 源文件后缀名，使其失效
3. 使光盘 yum 源生效
    + baseurl= #地址为光盘挂载地址
    + enalbled=1

## yum 命令

### 查询

**yum list ：查询所有可用软件包列表**

**yum search 关键字 ：搜索服务器上所有和关键字相关的包**

### 安装

**yum -y install 包名**
+ -install ：安装
+ -y ：自动回答 yes

### 升级

yum -y update 包名
+ update ：升级
+ -y ：自动回答 yes

### 卸载

yum -y remove 包名
+ remove ：卸载
+ -y ：自动回答 yes

**服务器使用最小化安装，用什么软件安装什么，尽量不卸载**

### yum 软件组管理命令

**yum grouplist ：列出所有可用的软件组列表**

**yum groupinstall 软件组名 ：安装指定软件组，组名可以由 grouplist 查询出来**

**yum groupmove 软件组名 ：卸载指定软件组**

注：软件组名必须为英文


# 源码包管理

源码包和 rpm 包的区别：

- [x] 安装之前的区别：概念上的区别
- [x] 安装之后的区别：安装位置不同
    + rpm 包是安装在默认位置中
    + 源码包安装在指定位置当中，一般是 /usr/local/软件名/ (源码包没有卸载命令)

安装位置不同带来的影响：

+ rpm 包安装的服务可以使用系统服务管理命令(service)来管理，例如 rpm 包安装的 apache 的启动方法是：
    + /ect/rc.d/init.d/httpd start
    + service httpd start
+ 源码包安装的服务不能被服务管理命令管理，因为没有安装到默认路径中。所以只能使用绝对路径进行服务的管理，如：
    + /usr/local/apache2/bin/apachectl start


## 源码包安装

安装准备：安装 C 语言编译器

注：windows 向 Linux 上传文件使用 WinSCP

安装注意事项：

1. 源代码保存位置：/usr/local/src/
2. 软件安装位置：/usr/local/
3. 如何确定安装过程报错：
    + 安装过程停止
    + 并出现 error, warning 或 no 的提示

源码包安装过程：

1. 下载源码包
2. 解压缩下载的源码包
3. 进入解压缩目录
4. ./configure 软件配置与检查
    + 定义需要的功能选项
    + 检测系统环境是否符合安装要求
    + 把定义好的功能选项和检测系统环境的信息都写入 Makefile 文件，用于后续的编辑
5. make #编译(如果报错执行 make clean, 重新执行 make)
6. make install #编译安装


# 脚本安装包

[lnmp.org](https://lnmp.org)
