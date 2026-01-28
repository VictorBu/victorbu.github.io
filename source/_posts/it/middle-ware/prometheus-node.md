---
title: Prometheus 使用之 node exporter
date: 2020-10-30 18:00:00
updated: 2020-10-30 18:00:00
categories: [IT]
tags: [Prometheus, node exporter, Grafana]
---

> 本文使用的 Prometheus 版本为 2.22.0，node exporter 版本为 1.0.1；部署在 Linux 服务器

Prometheus 是开源的监控报警系统和时序列数据库 (TSDB)；node exporter 用来监控服务器CPU、内存、磁盘、I/O等信息

# 一、node exporter

node exporter 下载地址：https://prometheus.io/download/#node_exporter，下载 node_exporter-1.0.1.linux-amd64.tar.gz

执行命令：

```
tar -xzvf node_exporter-1.0.1.linux-amd64.tar.gz

nohup ./xx/node_exporter-1.0.1.linux-amd64/node_exporter >/dev/null 2>&1 &
````

其中 xx 为自定义目录

默认端口为 9100

# 二、Prometheus

Prometheus 下载地址：https://prometheus.io/download/#prometheus，下载 prometheus-2.22.0.linux-amd64.tar.gz

执行命令：

```
tar -xzvf prometheus-2.22.0.linux-amd64.tar.gz
```

在 prometheus.yml 文件末尾添加：

```
scrape_configs:
  ... ...
  - job_name: 'node'
    static_configs:
    - targets: ['localhost:9100']
```

启动 Prometheus：

```
nohup /xx/prometheus-2.22.0.linux-amd64/prometheus \
--config.file=/xx/prometheus-2.22.0.linux-amd64/prometheus.yml \
--storage.tsdb.path=/xx/data/prometheus \
--storage.tsdb.retention=7d \
>/dev/null 2>&1 &
```

其中 xx 为自定义目录

默认端口为 9090

# 三、在 Grafana 中添加数据源与看板

## 3.1. 添加数据源

1. 登录 Grafana，在 Configuration > Data Sources 点击 "Add data source" 按钮，选中 Prometheus
1. URL 填入 http://localhost:9090，并保存

## 3.2. 添加看板

1. 在 "+" 点击 "Import"
1. 在 "Import via grafana.com" 下面的输入框，输入 8919，然后点击"Load"按钮
1. 在 Dashboards 即可看到面板列表

关于 8919 的解释：

Grafana 官网有已经可以直接使用的 dashboard，地址：https://grafana.com/grafana/dashboards 输入对应的 id 即可添加