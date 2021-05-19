---
title: 阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (五) Kubernetes TCP Ingress
date: 2021-05-18 10:00:00
updated: 2021-05-18 10:00:00
categories: [IT]
tags: [Alibaba, Alibaba Cloud, Kubernetes]
---

在 Kuberetes 应用中，一般是通过 Ingress 暴露 HTTP/HTTPS 的服务，但实际使用中可能需要暴露 TCP 服务。Ingress 默认包含了 Nginx，Nginx 本身支持 TCP 做反向代理，所以也 可以通过 Ingress 暴露 TCP 服务。

假设我们需要将服务 test-tcp 的 8081 端口暴露为 18081 端口：

# 一、修改配置项：tcp-services

切换到 kube-system 命名空间，选中 tcp-services 配置项，添加配置：

```
18081 default/test-tcp:8081
```

其中配置项的格式为：

```
<Nginx port>: <namespace/service name>:<service port>:[PROXY]:[PROXY]
```

# 二、修改服务：nginx-ingress-lb

切换到 kube-system 命名空间，选中 nginx-ingress-lb 服务，添加端口映射：

```
服务端口：18081 # 需要暴露的端口
容器端口：18081 # 第一步配置的端口，也可以配置成其他端口，只要与第一步保持一致即可
```

参考：

1. [玩转Kubernetes TCP Ingress](https://developer.aliyun.com/article/603225)
1. [Exposing TCP and UDP services](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services)