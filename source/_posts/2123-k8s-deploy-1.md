---
title: k8s 部署 (一) SkyWalking 集群
date: 2021-05-19 17:00:00
updated: 2021-05-19 17:00:00
categories: [IT]
tags: [Kubernetes, SkyWalking, Nacos, Elasticsearch]
---

本文部署的 SkyWalking 版本为 8.5.0，集群模式为 Nacos，存储使用 Elasticsearch 7

下载对应版本的源码并解压，如本文对应的为：[v8.5.0 for H2/MySQL/TiDB/InfluxDB/ElasticSearch 7](https://www.apache.org/dyn/closer.cgi/skywalking/8.5.0/apache-skywalking-apm-es7-8.5.0.tar.gz)

# 一、部署 OAP Server

## 1.1. 添加 ConfigMap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: skywalking-cm
data:
  CLUSTER: 'nacos'
  CLUSTER_NACOS_HOST_PORT: 'nacos-headless:8848'
  STORAGE: 'elasticsearch7'
  STORAGE_ES_CLUSTER_NODES: 'IP:PORT'
  ES_USER: '用户名'
  ES_PASSWORD: '密码'
  CORE_GRPC_PORT: '11800'
  CORE_REST_PORT: '12800'
```

配置项中为需要配置的环境变量，更多的环境变量配置见：config/application.yml

其中 nacos-headless:8848 为 Nacos 地址，根据需要修改；gRPC 端口为 agent 上传数据端口；rest 端口为 UI 调用端口

## 1.2. 添加 Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: skywalking
  name: skywalking
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: skywalking
  template:
    metadata:
      labels:
        app: skywalking
    spec:
      containers:
        - envFrom:
          - prefix: SW_
            configMapRef: 
              name: skywalking-cm                  
          image: apache/skywalking-oap-server:8.5.0-es7
          imagePullPolicy: IfNotPresent         
          name: skywalking
          ports:
            - containerPort: 12800
              name: http
              protocol: TCP
            - containerPort: 11800
              name: grpc
              protocol: TCP
          resources:
            limits:
              cpu: '2'
              memory: 2Gi
            requests:
              cpu: '1'
              memory: 2Gi
          volumeMounts:
            - mountPath: /etc/localtime
              name: volume-localtime
      volumes:
        - hostPath:
            path: /etc/localtime
            type: ''
          name: volume-localtime
```

## 1.3. 添加 Service

```
apiVersion: v1
kind: Service
metadata:
  name: skywalking
  labels:
    app: skywalking
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 12800
      protocol: TCP
      targetPort: 12800
    - name: grpc
      port: 11800
      protocol: TCP
      targetPort: 11800
  selector:
    app: skywalking
```

# 二、部署 UI

## 2.1. 添加 Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: skywalking-ui
  name: skywalking-ui
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: skywalking-ui
  template:
    metadata:
      labels:
        app: skywalking-ui
    spec:
      containers:
        - env:
            - name: SW_OAP_ADDRESS
              value: "skywalking:12800"          
          image: apache/skywalking-ui:8.5.0
          imagePullPolicy: IfNotPresent         
          name: skywalking-ui
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          resources:
            limits:
              cpu: '2'
              memory: 1Gi
            requests:
              cpu: '1'
              memory: 1Gi
          volumeMounts:
            - mountPath: /etc/localtime
              name: volume-localtime
      volumes:
        - hostPath:
            path: /etc/localtime
            type: ''
          name: volume-localtime
```

更多环境变量配置见：https://hub.docker.com/r/apache/skywalking-ui

## 2.2. 添加 Service

```
apiVersion: v1
kind: Service
metadata:
  name: skywalking-ui
  labels:
    app: skywalking-ui
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: skywalking-ui
```

# 三、程序改造

有两种设置 agent 的方法：

1. 将 agent 与程序打包在同一镜像中：实现简单
1. 使用 Kubernetes 的 Sidecar：更加灵活

本文使用第一种方法：

1. 修改 Dockerfile：

```
FROM 基础镜像
MAINTAINER VictorBu <VictorBu.xx@gmail.com>

ADD agent /agent
ADD *.jar /app.jar
ENTRYPOINT ["java", "-javaagent:/agent/skywalking-agent.jar", "-jar", "-server", "/app.jar"]
```

2. 部署时需要添加环境变量：

```
SW_AGENT_NAME: 对应程序的名字
SW_AGENT_COLLECTOR_BACKEND_SERVICES: skywalking:11800
```

更多的环境变量配置见：agent/config/agent.config




