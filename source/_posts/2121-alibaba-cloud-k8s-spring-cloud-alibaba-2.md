---
title: 阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (二) 部署微服务程序
date: 2021-03-22 12:00:00
updated: 2021-03-22 12:00:00
categories: [IT]
tags: [Alibaba, Alibaba Cloud, Spring Cloud, Kubernetes, Docker]
---

# 零、镜像

## 0.1. 母镜像选择

Alpine Linux 是一个面向安全应用的轻量级 Linux 发行版，基于 musl libc 和 busybox。Alpine 只有 5 M 左右，远远小于 CentOS 或 Ubuntu。

因为程序基于 Java 开发，所以微服务镜像需要 Java 1.8 的运行时环境支撑。项目选取 Oracle JDK 作为 Java 的运行时环境。最终使用的母镜像为：[frolvlad/alpine-oraclejre8:slim](https://github.com/Docker-Hub-frolvlad/docker-alpine-oraclejre8)

## 0.2. 基础镜像

Alpine 默认时区是 UTC 时间，需要修改时区，即制作自己的基础镜像：

```
FROM frolvlad/alpine-oraclejre8:slim
MAINTAINER VictorBu <VictorBu.xx@gmail.com>

RUN apk add tzdata \
	&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apk del tzdata
```

构建镜像并根据阿里云容器镜像服务将镜像上传，示例：

```
docker build -t alpine-oraclejre8-base:1.0 .

docker tag alpine-oraclejre8-base:1.0 registry.cn-shenzhen.aliyuncs.com/你的命名空间/alpine-oraclejre8-base:1.0

docker push registry.cn-shenzhen.aliyuncs.com/你的命名空间/alpine-oraclejre8-base:1.0
```

## 0.3. 业务镜像

示例（假设基础镜像的镜像名为：alpine-oraclejre8-base:1.0）

```
FROM alpine-oraclejre8-base:1.0
MAINTAINER VictorBu <VictorBu.xx@gmail.com>

ADD demo-0.0.1-SNAPSHOT.jar /demo.jar
ENTRYPOINT ["java", "-jar", "/demo.jar"]
```

构建镜像并根据阿里云容器镜像服务将镜像上传

# 一、部署微服务程序

工作负载->无状态->点击右上角”使用镜像创建“，傻瓜式操作，根据界面提示一步步操作即可，然后再根据需要添加服务或者路由。




