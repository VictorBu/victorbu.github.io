---
title: 浅尝 Elastic Stack (一) Elasticsearch、Kibana、Beats 安装
date: 2020-06-01 12:00:00
updated: 2020-06-01 12:00:00
categories: [IT]
tags: [Elastic, Elasticsearch, Kibana, Beats]
---

Elastic Stack 包括 Elasticsearch、Kibana、Beats 和 Logstash，也称为 ELK Stack。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。

Elastic 产品生态：

![](https://oss.x8y.cc/blog-img/2111/20190919085503769.png)

Elastic 协同：

![](https://oss.x8y.cc/blog-img/2111/20200223100338579.png)

推荐架构：

![](https://oss.x8y.cc/blog-img/2111/v2-3b578fddb1221bcdb250fcfdf3b070a2_r.jpg)


> 前置事项

1. 本系列使用 Elastic 7.7.0，操作系统为 CentOS 7.4
1. Elasticsearch 等不允许使用 root 启动，需要使用其他用户操作

#  一、安装 Elasticsearch

## 1.1 下载

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.7.0-linux-x86_64.tar.gz

tar -xzf elasticsearch-7.7.0-linux-x86_64.tar.gz
```

## 1.2 启动

```
cd elasticsearch-7.7.0/

# 后台运行
./bin/elasticsearch -d -p pid
```

## 1.3 查看是否成功启动

```
curl -X GET "localhost:9200/?pretty"
```

有类似如下输出，代码 Elasticsearch 成功启动：

```
{
  "name" : "localhost.localdomain",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "O64VFTnSSMyMx8Wc0U4nFQ",
  "version" : {
    "number" : "7.7.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "81a1e9eda8e6183f5237786246f6dced26a10eaf",
    "build_date" : "2020-05-12T02:01:37.602180Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```

## 关闭命令(根据需要执行)

```
pkill -F pid
```

# 二、安装 Kibana

## 2.1 下载

```
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.7.0-linux-x86_64.tar.gz
tar -xzf kibana-7.7.0-linux-x86_64.tar.gz
```

## 2.2 设置中文页面(根据需要执行)

/config/kibana.yml

```
i18n.locale: "zh-CN"
```

## 2.3 启动

```
cd kibana-7.7.0-darwin-x86_64/

# 后台运行
nohup ./bin/kibana &
```

Kibana 默认启动 5601 端口，输入 IP:5601 即可访问


# 三、Beats

可以根据不同的需要安装不同的 beat，可以在 Kibana 页面按照指引操作：

![](https://oss.x8y.cc/blog-img/2111/kibanaUI.png)

> 参考

1. [Elastic：菜鸟上手指南](https://elasticstack.blog.csdn.net/article/details/102728604)
1. [Lateautunm4lin的回答](https://www.zhihu.com/question/54058964/answer/868223041)
1. [如何在Linux，MacOS及Windows上进行安装Elasticsearch](https://blog.csdn.net/UbuntuTouch/article/details/99413578)
1. [Kibana：如何在Linux，MacOS及Windows上安装Elastic栈中的Kibana](https://elasticstack.blog.csdn.net/article/details/99433732)
