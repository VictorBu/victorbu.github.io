---
title: 阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (一) 部署 Nacos
date: 2021-03-19 12:00:00
updated: 2021-03-19 12:00:00
categories: [IT]
tags: [Alibaba, Alibaba Cloud, Spring Cloud, Kubernetes, Nacos]
---

# 零、准备工作

## 0.1. 说明

1. 目前官网提供的最新镜像的 Nacos 版本为 1.4.1，但是在部署过程中有问题，实际使用为 1.3.0
1. 官方文档提供了自动伸缩的部署方式，但需要部署持久卷声明（PersistentVolumeClaim 简称 PVC），故目前仍采用固定数量的部署方式
1. 官方文档使用的数据库是自己部署的，因为实际中使用阿里云，故直接使用阿里云 RDS （阿里云 k8s 部署的程序可以通过内网 IP 直接访问阿里云其他服务，如 RDS, Redis, Kafka 等）

## 0.2. 连接 Kubernetes 集群配置

如果需要在集群外的服务器执行 kubectl 命令，需要先进行连接 Kubernetes 集群配置：

![](https://oss.x8y.cc/blog-img/2121/1/kubectl.png)

# 一、开始部署 Nacos


## 1.1. 初始化数据库

下载 [Nacos 1.3.0](https://github.com/alibaba/nacos/releases/download/1.3.0/nacos-server-1.3.0.zip) ，然后在数据库执行 conf/nacos-mysql.sql 初始化数据库

## 1.2. 部署 Nacos

克隆 nacos-k8s 项目：```git clone https://github.com/nacos-group/nacos-k8s.git```

修改 ./deploy/nacos/nacos-quick-start.yaml 文件：

```
---
apiVersion: v1
kind: Service
metadata:
  name: nacos-headless
  labels:
    app: nacos-headless
spec:
  type: ClusterIP
  clusterIP: None
  ports:
    - port: 8848
      name: server
      targetPort: 8848
    - port: 7848
      name: rpc
      targetPort: 7848
  selector:
    app: nacos
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nacos-cm
data:
  mysql.host: "阿里云 RDS 内网地址"
  mysql.db.name: "数据库名"
  mysql.port: "数据库端口"
  mysql.user: "数据库用户名"
  mysql.password: "数据库密码"
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nacos
spec:
  serviceName: nacos-headless
  replicas: 3 # 实例数量
  template:
    metadata:
      labels:
        app: nacos
      annotations:
        pod.alpha.kubernetes.io/initialized: "true"
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                      - nacos
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: k8snacos
          imagePullPolicy: Always
          image: nacos/nacos-server:1.3.0 # nacos 版本
          resources:
            requests:
              memory: "2Gi"
              cpu: "500m"
          ports:
            - containerPort: 8848
              name: client
            - containerPort: 7848
              name: rpc
          env:
            - name: NACOS_REPLICAS
              value: "3" # 实例数量
            - name: MYSQL_SERVICE_HOST
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.host               
            - name: MYSQL_SERVICE_DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.db.name
            - name: MYSQL_SERVICE_PORT
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.port
            - name: MYSQL_SERVICE_USER
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.user
            - name: MYSQL_SERVICE_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.password
            - name: NACOS_SERVER_PORT
              value: "8848"
            - name: NACOS_APPLICATION_PORT
              value: "8848"
            - name: PREFER_HOST_MODE
              value: "hostname"
            - name: NACOS_SERVERS
              value: "nacos-0.nacos-headless.default.svc.cluster.local:8848 nacos-1.nacos-headless.default.svc.cluster.local:8848 nacos-2.nacos-headless.default.svc.cluster.local:8848" # 如果调整了数量或者命名空间这里也需要调整
  selector:
    matchLabels:
      app: nacos
```

在 nacos-k8s 根目录执行：```kubectl create -f ./deploy/nacos/nacos-quick-start.yaml```，刷新页面可以看到部署结果：

![](https://oss.x8y.cc/blog-img/2121/1/nacos.png)


> 参考：

1. [nacos-k8s](https://github.com/nacos-group/nacos-k8s)
1. [nacos-docker](https://github.com/nacos-group/nacos-docker)
1. [nacos](https://github.com/alibaba/nacos)
