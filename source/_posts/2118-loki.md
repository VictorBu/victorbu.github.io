---
title: 日志聚合工具之 Loki
date: 2020-09-27 20:00:00
updated: 2020-09-17 20:00:00
categories: [IT]
tags: [Loki, Promtail, Grafana]
---

> 本文使用的 Loki 和 Promtail 版本为 1.6.1，Grafana 版本为 7.2.0；部署在 Linux 服务器

Loki 负责日志的存储和查询；Promtail 负责日志的采集并推送给 Loki；Grafana 负责日志展示

# 一、Loki

Loki 下载地址：https://github.com/grafana/loki/releases，下载 loki-linux-amd64.zip

在 Loki 的源码里找到对应版本的配置文件：/cmd/loki/loki-local-config.yaml，文件内容为：

```
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
    - from: 2018-04-15
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /tmp/loki/index

  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```

根据具体情况决定是否修改上述配置，然后执行以下命令：

```
unzip loki-linux-amd64.zip

nohup ./loki-linux-amd64 -config.file=loki-local-config.yaml >/dev/null 2>&1 &
```

# 二、Promtail

Promtail 下载地址：https://github.com/grafana/loki/releases，下载 promtail-linux-amd64.zip

在 Loki 的源码里找到对应版本的配置文件：/cmd/promtail/promtail-local-config.yaml，文件内容为：

```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
```

根据具体情况决定是否修改上述配置或者添加 job，然后执行以下命令：

```
unzip promtail-linux-amd64.zip

nohup ./promtail-linux-amd64 -config.file=promtail-local-config.yaml >/dev/null 2>&1 &
```

# 三、Grafana

## 3.1. 下载并启动

如果之前没有安装过 Grafana，需要先下载安装，下载地址：https://grafana.com/grafana/download

执行以下命令：

```
tar -xzvf grafana-7.2.0.linux-amd64.tar.gz

cd grafana-7.2.0

nohup ./bin/grafana-server >/dev/null 2>&1 &
```

如果不是在 grafana 所在目录执行运行命令需要添加参数，如：-homepath /xx/grafana-7.2.0

其中 xx 为自定义目录

默认端口为 3000，用户名密码均为 admin

## 3.2. 配置

1. 登录 Grafana，在 Configuration > Data Sources 点击 "Add data source" 按钮，选中 Loki
1. URL 填入 http://localhost:3100，并保存
1. 在 Explore 中选中上面添加的 Loki 数据源 既可以看到日志信息

