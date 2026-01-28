---
title: 阿里云 k8s 部署 Spring Cloud Alibaba 微服务实践 (三) 服务观测
date: 2021-03-22 15:00:00
updated: 2021-05-18 09:00:00
categories: [IT]
tags: [Alibaba, Alibaba Cloud, Spring Cloud, Kubernetes, Prometheus, Grafana]
---

> 可观测性指如何从外部输出推断及衡量系统内部状态。Kubernetes 可观测性体系包含监控和日志两部分，监控可以帮助开发者查看系统的运行状态，而日志可以协助问题的排查和诊断从可观测性的角度，以 ACK（阿里云 Kubernetes） 为基础的系统架构可以粗略分为 4 个层次。自下而上分别是：基础设施层、容器性能层、应用性能层、用户业务层。

直接使用阿里云提供的：阿里云托管版 Prometheus（ARMS Prometheus）、阿里云日志服务 SLS （Log Service），可以无需自行搭建 GPE、EFK 等。

# 一、日志管理（采集程序日志）

可直接参考：[通过日志服务采集Kubernetes容器日志](https://help.aliyun.com/document_detail/87540.html?spm=a2c4g.11186623.6.897.447e41aaWLKqUX)

启用日志服务组件 Logtail 后，在第二步“容器配置”中的“日志配置”可以启用日志：

![](https://oss.x8y.cc/blog-img/2121/3/log-1.png)

根据提示操作即可，例如直接采集容器内日志可以在路径处填入：stdout，效果如下：

![](https://oss.x8y.cc/blog-img/2121/3/log-2.png)

# 二、监控管理（采集程序性能指标）

可直接参考：[阿里云Prometheus监控](https://help.aliyun.com/document_detail/161304.html?spm=a2c4g.11186623.6.907.6a43388fp6ZAjR)

开启阿里云 Prometheus 监控后，在第三步“高级配置”中的“标签和注解”中添加 POD 注解：

![](https://oss.x8y.cc/blog-img/2121/3/prometheus.png)

**注意：是在 POD 添加注解，而不是在 Deployment 添加注解，官方文档没有特别说明，在此浪费了不少时间**

在“运维管理”中的“Prometheus 监控”点击右上角的“在新页面打开”即可打开 Grafana 页面：在 “+” 点击 “Import”，在 “Import via grafana.com” 下面的输入框，输入 4701，然后点击”Load”按钮：

![](https://oss.x8y.cc/blog-img/2121/3/grafana.png)

*以下内容 2021/05/18 更新*

阿里云日志服务默认不支持中文搜索，需要手动开启：

“日志服务”-“选中对应的 Project”-“选中对应的 logstore”-右上角“查询分析属性”-“属性”-开启“包含中文”



> 参考

1. [可观测性体系概述](https://help.aliyun.com/document_detail/203344.html?spm=a2c4g.11186623.6.897.447e41aaAP1rwH)
1. [通过日志服务采集Kubernetes容器日志](https://help.aliyun.com/document_detail/87540.html?spm=a2c4g.11186623.6.897.447e41aaWLKqUX)
1. [阿里云Prometheus监控](https://help.aliyun.com/document_detail/161304.html?spm=a2c4g.11186623.6.907.6a43388fp6ZAjR)
