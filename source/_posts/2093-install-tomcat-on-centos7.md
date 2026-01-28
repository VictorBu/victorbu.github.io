---
title: 在 CentOS 7 安装 Tomcat
date: 2019-09-18 15:30:00
updated: 2019-09-18 15:30:00
categories: [IT]
tags: [Tomcat, JDK, Linux]
---

# 一、 安装 JDK 8

## 1.1 下载 JDK 8

```
cd /opt/

wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
```

## 1.2 安装 JDK 8

```
tar xzf jdk-8u141-linux-x64.tar.gz

cd jdk1.8.0_141

alternatives --install /usr/bin/java java /opt/jdk1.8.0_141/bin/java 2

alternatives --config java
```

界面显示内容如下：

```
There is 1 program that provides 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           /opt/jdk1.8.0_141/bin/java

Enter to keep the current selection[+], or type selection number: 
```

输入 1 回车

## 1.3 设置  javac 和 jar 命令的路径

```
alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_141/bin/jar 2
alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_141/bin/javac 2
alternatives --set jar /opt/jdk1.8.0_141/bin/jar
alternatives --set javac /opt/jdk1.8.0_141/bin/javac
```

## 1.4 查看是否正确安装

```
java -version
```

## 1.5 设置环境变量

```
export JAVA_HOME=/opt/jdk1.8.0_141
export JRE_HOME=/opt/jdk1.8.0_141/jre
export PATH=$PATH:/opt/jdk1.8.0_141/bin:/opt/jdk1.8.0_141/jre/bin
```
# 二、安装 Tomcat

## 2.1 下载 Tomcat

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.45/bin/apache-tomcat-8.5.45.tar.gz

```

## 2.2 安装 Tomcat

```
tar xzf apache-tomcat-8.5.45.tar.gz

mv apache-tomcat-8.5.45 /usr/local/tomcat8
```

## 2.3 启动 Tomcat 

```
/usr/local/tomcat8/bin/startup.sh
```

注意关闭防火墙或放行端口，输入：ip:8080



> 参考：

1. [How To Install Java JDK 8 on CentOS 7](https://idroot.us/how-to-install-java-jdk-8-on-centos-7/)
1. [wget下载jdk1.8](https://www.liangzl.com/get-article-detail-6953.html)

