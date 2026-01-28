---
title: Jenkins 安装及配置
date: 2021-03-31 10:00:00
updated: 2021-03-31 10:00:00
categories: [IT]
tags: [Jenkins]
---

本文使用 CentOS 7

1. 安装 JDK
本文使用 rpm 安装

https://download.oracle.com/otn-pub/java/jdk/8u281-b09/89d678f2be164786b292527658ca1605/jdk-8u281-linux-x64.rpm

```rpm -ivh jdk-8u281-linux-x64.rpm```

无需配置环境变量

默认下载链接需要登录，将链接中的 otn 改为 otn-pub 即可免登录下载，见：[Oracle 免登录下载 JDK](https://blog.csdn.net/qq_52859790/article/details/113740195)

1. 安装 Maven
下载：https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

```tar -zxvf  apache-maven-3.6.3-bin.tar.gz```

```
# vi /etc/profile

MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3
export PATH=${MAVEN_HOME}/bin:${PATH}

# source /etc/profile
# mvn -v
```

可选：替换 maven 源、指定包下载位置等

1. 安装 git

```yum install -y git```

1. 安装 docker

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl start docker

systemctl enable docker
```



1. 安装 kubectl

```curl -Lo kubectl http://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubectl && chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl```



安装 jenkins

方法一：

```
curl -Lo jenkins-2.277.1-1.1.noarch.rpm https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.277.1-1.1.noarch.rpm

rpm -ivh jenkins-2.277.1-1.1.noarch.rpm

systemctl start jenkins

systemctl enable jenkins
```


方法二：

```
curl -Lo /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins

systemctl start jenkins

systemctl enable jenkins
```

记得放行

```cat /var/lib/jenkins/secrets/initialAdminPassword```


参考：

[Jenkins 使用国内镜像快速安装（rpm安装）](https://blog.csdn.net/qiuyeyijian/article/details/104570642)