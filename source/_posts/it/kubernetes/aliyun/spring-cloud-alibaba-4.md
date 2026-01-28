---
title: 阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (四) 自动化部署
date: 2021-03-30 10:00:00
updated: 2021-05-08 19:00:00
categories: [IT]
tags: [Alibaba, Alibaba Cloud, Kubernetes, Jenkins, JDK, Maven, Git, Docker, Node.js]
---

本文使用操作系统为 CentOS 7，Jenkins 版本为 2.277.1

# 零、准备工作

## 0.1. 安装 JDK

本文使用 rpm 安装：[下载 rpm 包](https://download.oracle.com/otn-pub/java/jdk/8u281-b09/89d678f2be164786b292527658ca1605/jdk-8u281-linux-x64.rpm)

```
rpm -ivh jdk-8u281-linux-x64.rpm
```

注：

1. 无需配置环境变量
1. 默认下载链接需要登录，将链接中的 otn 改为 otn-pub 即可免登录下载，见：[Oracle 免登录下载 JDK](https://blog.csdn.net/qq_52859790/article/details/113740195)

## 0.2. 安装配置 Maven

本文 Maven 安装在：/usr/local/maven，下载 Maven：[下载链接](https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)


```
tar -zxvf  apache-maven-3.6.3-bin.tar.gz
```

添加环境变量：

```
# vi /etc/profile

MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3
export PATH=${MAVEN_HOME}/bin:${PATH}

# source /etc/profile
# mvn -v
```

注：根据实际情况决定是否替换 Maven 源、指定包下载位置等

## 0.3. 安装 Git

```
yum install -y git
```

## 0.4. 安装配置 Docker

本文使用阿里云镜像：

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

systemctl start docker

systemctl enable docker
```

注：需登录阿里云容器镜像服务

## 0.5 安装配置 kubectl

本文使用阿里云地址：

```
curl -Lo kubectl http://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.20.0/bin/linux/amd64/kubectl && chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl
```

如果需要在集群外的服务器执行 kubectl 命令，需要先进行连接 Kubernetes 集群配置：

![](https://oss.x8y.cc/blog-img/2121/1/kubectl.png)

# 一、安装配置 Jenkins

## 1.1. 方法一（本文使用）

使用清华大学 Jenkins 镜像：

```
curl -Lo jenkins-2.277.1-1.1.noarch.rpm https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat-stable/jenkins-2.277.1-1.1.noarch.rpm

rpm -ivh jenkins-2.277.1-1.1.noarch.rpm

systemctl start jenkins

systemctl enable jenkins
```


## 1.2. 方法二（未实际测试）

```
curl -Lo /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins

systemctl start jenkins

systemctl enable jenkins
```

## 1.3. 初始化

Jenkins 默认端口为 8080（记得放行），可以根据需要修改，亦可修改其他配置，详见：[Jenkins 使用国内镜像快速安装（rpm 安装）](https://blog.csdn.net/qiuyeyijian/article/details/104570642)

1. 访问：IP:8080，第一次访问需要输入密码，查看密码：```cat /var/lib/jenkins/secrets/initialAdminPassword```
1. 设置管理员账号
1. 安装默认插件

## 1.4. 配置

因为实际测试中 Maven Integration 使用有问题，所以未使用该插件，因此不需要配置 Maven，只需配置 JDK 和 Git 即可：“系统管理”--“全局工具配置”

![](https://oss.x8y.cc/blog-img/2121/4/jdk.png)

注：使用 rpm 安装 JAVA_HOME为：/usr/java/default

![](https://oss.x8y.cc/blog-img/2121/4/git.png)

注：因 Jenkins 默认执行用户为 jenkins，会有一些权限问题，所以本文修改执行用户为 root：

```
# vi /etc/sysconfig/jenkins

JENKINS_USER="root"

# systemctl restart jenkins
```


# 二、使用 Jenkins 自动化部署

## 2.1. 常规部署（适用于测试环境）

1. 新建任务
1. 构建一个自由风格的软件项目
1. 配置项目 Git 地址、认证、分支等
1. “构建”--“执行 shell”：

```
## 0.删除自己构建的包，防止有更新没有获取到
## rm -rf /usr/local/maven/repository/com/YOUR_COMPANY/*

## 1.打包项目
/usr/local/maven/apache-maven-3.6.3/bin/mvn clean package -Dmaven.test.skip=true

## 2.添加 docker 相关文件
targetDir="/var/project/$JOB_BASE_NAME"
if [ ! -d "$targetDir" ]; then
        mkdir $targetDir
fi

cp -f ./target/*.jar $targetDir/

cp -f /var/project/Dockerfile $targetDir/

cd $targetDir

# 3.构建镜像、推送
docker build -t registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:latest .
docker push registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:latest

# 4.升级
##kubectl set image deployment $JOB_BASE_NAME $JOB_BASE_NAME=registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:latest
/usr/local/bin/kubectl rollout restart deployment $JOB_BASE_NAME ## 2021/05/08
```

## 2.2. 通过 tag 标签部署（适用于生产环境）

1. 安装 Git Parameter 插件
2. 新建任务
3. 构建一个自由风格的软件项目
4. 配置：

![](https://oss.x8y.cc/blog-img/2121/4/tag.png)

![](https://oss.x8y.cc/blog-img/2121/4/git-tag.png)

5. “构建”--“执行 shell”：

```
## 如果是回滚，不需要打包、构建镜像等操作，直接执行升级命令
if [ $ROLLBACK -eq 0 ]; then

## 0.删除自己构建的包，防止有更新没有获取到
## rm -rf /usr/local/maven/repository/com/YOUR_COMPANY/*

## 1.打包项目
/usr/local/maven/apache-maven-3.6.3/bin/mvn clean package -Dmaven.test.skip=true

## 2.添加 docker 相关文件
targetDir="/var/project/$JOB_BASE_NAME"
if [ ! -d "$targetDir" ]; then
        mkdir $targetDir
fi

cp -f ./target/*.jar $targetDir/

cp -f /var/project/Dockerfile $targetDir/

cd $targetDir

# 3.构建镜像、推送
docker build -t registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:$TAG .
docker push registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:$TAG

fi

# 4.升级
kubectl set image deployment $JOB_BASE_NAME $JOB_BASE_NAME=registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:$TAG
```

*以下内容 2021/05/08 更新*

## 2.3. 部署前端(Vue.js)项目（常规部署）

如果没有 Node.js 则需安装 Node.js 和 cnpm ：

```
cd /usr/local/
tar -xvf node-v14.16.1-linux-x64.tar.xz
mv node-v14.16.1-linux-x64/ nodejs

ln -s /usr/local/nodejs/bin/npm /usr/bin/npm
ln -s /usr/local/nodejs/bin/node /usr/bin/node

npm install cnpm -g --registry=https://registry.npm.taobao.org
```

部署脚本：

```
## 1.打包项目
/usr/local/nodejs/bin/cnpm i
/usr/local/nodejs/bin/npm run build

## 2.添加 docker 相关文件
targetDir="/var/project/$JOB_BASE_NAME"
if [ ! -d "$targetDir" ]; then
        mkdir $targetDir
fi

rm -rf $targetDir/*
cp -f /var/project/Dockerfile $targetDir/
cp -rf dist $targetDir/dist

## 如果是公众号项目需要拷贝公众号的 txt 文件
## cp /var/project/XXXXXXXXX.txt  $targetDir/dist/XXXXXXXXX.txt

cd $targetDir

# 3.构建镜像、推送
docker build -t registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:latest .
docker push registry.cn-shenzhen.aliyuncs.com/YOUR_NAMESPACE/$JOB_BASE_NAME:latest

# 4.升级
/usr/local/bin/kubectl rollout restart deployment $JOB_BASE_NAME
```

其中 Dockerfile 内容：

```
FROM nginx:alpine
MAINTAINER VictorBu <VictorBu.xx@gmail.com>

ADD dist /usr/share/nginx/html
```