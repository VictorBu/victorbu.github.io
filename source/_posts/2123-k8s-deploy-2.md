---
title: k8s 部署 (二) EMQ X 集群
date: 2021-05-20 10:00:00
updated: 2021-05-20 10:00:00
categories: [IT]
tags: [Kubernetes, EMQ X, MQTT]
---

本文部署的 EMQ X Broker 版本为 4.3.1

# 一、RBAC 鉴权

集群需要使用到 Kubernetes 的 API Server，但是普通 Pod 是没有权限访问的，需要授权：

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: default
  name: emqx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: emqx
  namespace: default

rules:
  - apiGroups:
      - ''
    resources:
      - endpoints 
    verbs: 
      - get
      - watch
      - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: emqx
  namespace: default
roleRef:
  kind: Role
  name: emqx
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: emqx
    namespace: default
```

如果没有授权，会有如下报错：

```
Ekka(AutoCluster): Discovery error: {403,"{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"endpoints "emqx-headless" is forbidden: User "system:serviceaccount:default:default" cannot get resource "endpoints" in API group "" in the namespace "default"","reason":"Forbidden","details":{"name":"emqx-headless","kind":"endpoints"},"code":403}"}
```

# 二、新增 ConfigMap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: emqx-cm
data:
  NAME: "emqx"
  CLUSTER__DISCOVERY: "k8s"
  CLUSTER__K8S__ADDRESS_TYPE: "ip"
  CLUSTER__K8S__APISERVER: "https://IP:PORT"
  CLUSTER__K8S__NAMESPACE: "default"
  CLUSTER__K8S__SERVICE_NAME: "emqx-headless"
  CLUSTER__K8S__APP_NAME: "emqx"
```

默认情况下 EMQ X 使用带有 EMQX_ 的前缀的环境变量来覆盖配置文件中的配置项环境变量名称到配置文件键值名称映射规则如下:将 EMQX_ 前缀移除；大写字符替换成小写；双下划线 __ 替换成点 .  详见：[使用环境变量修改配置](https://docs.emqx.cn/broker/v4.3/configuration/environment-variable.html)

+ cluster.kubernetes.apiserver 为 kubernetes apiserver 的地址，可以通过 kubectl cluster-info 命令获取
+ cluster.kubernetes.service_name 为 Service 的 name
+ cluster.kubernetes.app_name 为 EMQ X Broker 的 node.name 中 @ 符号之前的部分，需要同时将集群中 EMQ X Broker 设置为统一的 node.name 的前缀

# 三、新增 Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: emqx
  name: emqx
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: emqx
  template:
    metadata:
      labels:
        app: emqx
    spec:
      serviceAccountName: emqx
      containers:
        - envFrom:
          - prefix: EMQX_
            configMapRef: 
              name: emqx-cm            
          image: emqx/emqx:4.3.1
          imagePullPolicy: IfNotPresent  
          livenessProbe:
            exec:
              command:
                - emqx_ctl
                - status
            failureThreshold: 3
            initialDelaySeconds: 60
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1          
          name: emqx
          ports:
            - name: mqtt
              protocol: TCP
              containerPort: 1883
            - name: mqttssl
              protocol: TCP
              containerPort: 8883
            - name: mgmt
              protocol: TCP
              containerPort: 8081
            - name: websocket
              protocol: TCP
              containerPort: 8083
            - name: wss
              protocol: TCP
              containerPort: 8084
            - name: dashboard
              protocol: TCP
              containerPort: 18083  
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

+ 1883	MQTT 协议端口
+ 8883	MQTT/SSL 端口
+ 8083	MQTT/WebSocket 端口
+ 8084	MQTT/WebSocket/SSL 端口
+ 8081	管理 API 端口
+ 18083	Dashboard 端口

# 四、新增 Service

```
apiVersion: v1
kind: Service
metadata:
  name: emqx-headless
  labels:
    app: emqx-headless
spec:
  type: ClusterIP
  clusterIP: None
  ports:
    - name: mqtt
      port: 1883
      protocol: TCP
      targetPort: 1883
    - name: mqttssl
      port: 8883
      protocol: TCP
      targetPort: 8883
    - name: mgmt
      port: 8081
      protocol: TCP
      targetPort: 8081
    - name: websocket
      port: 8083
      protocol: TCP
      targetPort: 8083
    - name: wss
      port: 8084
      protocol: TCP
      targetPort: 8084
    - name: dashboard
      port: 18083
      protocol: TCP
      targetPort: 18083      
  selector:
    app: emqx
```

# 五、放行 TCP 端口

见：[阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (五) Kubernetes TCP Ingress](https://www.cnblogs.com/victorbu/p/14780037.html)

参考：

1.[从零开始建立 EMQ X MQTT 服务器 的 K8S 集群](https://zhuanlan.zhihu.com/p/148734253)
1. [EMQ X Broker 文档](https://docs.emqx.cn/broker/v4.3/)