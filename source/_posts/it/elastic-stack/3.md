---
title: 浅尝 Elastic Stack (三) Logstash + Beats
date: 2020-06-03 09:00:00
updated: 2020-06-03 09:00:00
categories: [IT]
tags: [Elastic, Kibana, Logstash, Beats]
---


本文使用 Filebeat，如果没有安装需要安装：

```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.7.0-linux-x86_64.tar.gz
tar xzvf filebeat-7.7.0-linux-x86_64.tar.gz
```

# 一、Filebeat 配置

在 Filebeat 根目录新建文件 filebeat_logstash.yml ：


```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /project/logstash/weblog-sample.log
 
output.logstash:
  hosts: ["localhost:9900"]
```

# 二、Logstash 配置

修改配置文件 weblog.conf：

```
input {  
  beats {
    port => "9900"
  }
}


filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  
  
  geoip {
    source => "clientip"
  }
  
  useragent {
    source => "user_agent"
    target => "useragent"
  }
  
  date {
    match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
  }
}
 
output {
  file {
    path => "/project/logs/logstashtest.log"
    }
	
  elasticsearch {
    hosts => ["localhost:9200"]
	index => "weblog_elastic_example"
  }
}
```

# 三、运行

```
./filebeat -e -c filebeat_logstash.yml
```

总计有 300000 条日志数据，需要等待一段时间才能完成导入，可以通过 Kibana 在 Dev Tools 输入以下命令查看结果：

查看索引命令：

```
GET _cat/indices
```

count 命令：

```
GET weblog_elastic_example/_count
```

search 命令：


```
GET weblog_elastic_example/_search
```

> 参考

1. [Logstash：Logstash 入门教程 （二）](https://elasticstack.blog.csdn.net/article/details/105979677)
1. [Beats Directory Layout](https://www.elastic.co/guide/en/beats/filebeat/7.7/directory-layout.html)