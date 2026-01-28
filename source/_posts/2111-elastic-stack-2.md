---
title: 浅尝 Elastic Stack (二) Logstash
date: 2020-06-02 12:00:00
updated: 2020-06-02 12:00:00
categories: [IT]
tags: [Elastic, Logstash, Elasticsearch, Kibana]
---

# 一、安装与启动

Logstash 依赖 Java 8 或者 Java 11，需要先安装 JDK

## 1.1 下载

```
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.7.0.rpm
```

## 1.2 安装

```
sudo rpm -i logstash-7.7.0.rpm
```

Logstash 的目录结构见：[Directory Layout of Debian and RPM Packages](https://www.elastic.co/guide/en/logstash/7.7/dir-layout.html#deb-layout)

## 1.3 修改配置(根据需要执行)

修改 /etc/logstash/logstash.yml 配置：

```
config.reload.automatic : true
```

这样修改配置文件后，不需要重启 Logstash

## 1.4 启动

```
sudo systemctl start logstash.service
```

## 1.5 测试启动

```
cd /usr/share/logstash

sudo bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

然后输入：hello world，可以看到下面的输出：

```
{
      "@version" => "1",
          "host" => "localhost.localdomain",
       "message" => "hello world",
    "@timestamp" => 2020-05-29T23:16:52.686Z
}
```

# 二、使用

## 2.1 新建配置文件

```
cd /etc/logstash/conf.d/
vi weblog.conf
```

weblog.conf 的内容为：

```
input {
  tcp {
    port => 9900
  }
}
 
output {
  file {
    path => "/project/logs/logstashtest.log"
    }
}
```

配置文件的含义是监听 9900 端口的输入，并保存到 /project/logs/logstashtest.log

## 2.2 使用

```
echo 'hello logstash' | nc localhost 9900
````

查看 /usr/local/logstash/test.log 的内容，可以看到类似如下内容：

```
{
    "message":"hello logstash",
    "@timestamp":"2020-05-30T19:08:34.043Z",
    "host":"localhost",
    "port":47332,
    "@version":"1"
}
```

# 三、过滤器

先下载测试使用的数据：[weblog-sample.log](https://ela.st/weblog-sample)，内容是一个 log 文件，格式如下：

```
14.49.42.25 - - [12/May/2019:01:24:44 +0000] "GET /articles/ppp-over-ssh/ HTTP/1.1" 200 18586 "-" "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5"
```

## 3.1 grok

修改配置文件 weblog.conf：

```
input {
  tcp {
    port => 9900
  }
}


filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}
 
output {
  file {
    path => "/project/logs/logstashtest.log"
    }
}
```

%{COMBINEDAPACHELOG} 是 Logstash 自带的匹配模式，表达式为：

```
%{IPORHOST:clientip} %{USER:ident} %{USER:auth} 
\[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}
(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} 
(?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent}
```

读入 weblog-sample.log 的第一行数据：

```
head -n 1 weblog-sample.log | nc localhost 9900
```

得到输出类似如下：

```
{
    "request":"/articles/ppp-over-ssh/",
    "@timestamp":"2020-05-30T22:31:37.309Z",
    "port":47428,
    "host":"localhost",
    "timestamp":"12/May/2019:01:24:44 +0000",
    "response":"200",
    "referrer":"\"-\"",
    "ident":"-",
    "@version":"1",
    "verb":"GET",
    "clientip":"14.49.42.25",
    "message":"14.49.42.25 - - [12/May/2019:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\"",
    "auth":"-",
    "httpversion":"1.1",
    "bytes":"18586",
    "agent":"\"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
}

```

通过 grok 使用正则表达式将非结构化的数据转换为结构化的数据

Kibana 自带了 grok 调试工具，可以在 Dev Tools 中 Grok Debugger 调试

## 3.2 geoip

```
input {
  tcp {
    port => 9900
  }
}


filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  
  
  geoip {
    source => "clientip"
  }
}
 
output {
  file {
    path => "/project/logs/logstashtest.log"
    }
}
```

读入 weblog-sample.log 的第一行数据：

```
head -n 1 weblog-sample.log | nc localhost 9900
```

得到输出类似如下：

```
{
    "geoip":{
        "longitude":126.97409999999999,
        "ip":"14.49.42.25",
        "country_name":"South Korea",
        "country_code3":"KR",
        "country_code2":"KR",
        "location":{
            "lon":126.97409999999999,
            "lat":37.5112
        },
        "latitude":37.5112,
        "continent_code":"AS",
        "timezone":"Asia/Seoul"
    },
    "request":"/articles/ppp-over-ssh/",
    "@timestamp":"2020-05-30T22:44:17.084Z",
    "port":47436,
    "host":"localhost",
    "timestamp":"12/May/2019:01:24:44 +0000",
    "response":"200",
    "referrer":"\"-\"",
    "ident":"-",
    "@version":"1",
    "verb":"GET",
    "clientip":"14.49.42.25",
    "message":"14.49.42.25 - - [12/May/2019:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\"",
    "auth":"-",
    "httpversion":"1.1",
    "bytes":"18586",
    "agent":"\"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
}
```

geoip 将 IP 地址转换为地理位置等信息

## 3.3 useragent

```
input {
  tcp {
    port => 9900
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
    source => "agent"
    target => "useragent"
  }
}
 
output {
  file {
    path => "/project/logs/logstashtest.log"
    }
}
```

读入 weblog-sample.log 的第一行数据：

```
head -n 1 weblog-sample.log | nc localhost 9900
```

得到输出类似如下：

```
{
    "geoip":{
        "longitude":126.97409999999999,
        "ip":"14.49.42.25",
        "country_name":"South Korea",
        "country_code3":"KR",
        "country_code2":"KR",
        "location":{
            "lon":126.97409999999999,
            "lat":37.5112
        },
        "latitude":37.5112,
        "continent_code":"AS",
        "timezone":"Asia/Seoul"
    },
    "request":"/articles/ppp-over-ssh/",
    "@timestamp":"2020-05-30T22:58:17.848Z",
    "port":47444,
    "host":"localhost",
    "timestamp":"12/May/2019:01:24:44 +0000",
    "response":"200",
    "referrer":"\"-\"",
    "ident":"-",
    "useragent":{
        "minor":"6",
        "major":"3",
        "build":"",
        "device":"Other",
        "os_name":"Windows",
        "patch":"b1",
        "name":"Firefox Beta",
        "os":"Windows"
    },
    "@version":"1",
    "verb":"GET",
    "clientip":"14.49.42.25",
    "message":"14.49.42.25 - - [12/May/2019:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\"",
    "auth":"-",
    "httpversion":"1.1",
    "bytes":"18586",
    "agent":"\"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
}
```

useragent 解析浏览器及操作系统信息

## 3.4 date

Logstash 将事件时间存储在 @timestamp 字段中，但 weblog-sample.log 创建时间在 timestamp 字段中，该字段的格式不是 ISO8601，可以使用 date 过滤器将此字段转换为日期类型

```
input {
  tcp {
    port => 9900
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
    source => "agent"
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
}
```

读入 weblog-sample.log 的第一行数据：

```
head -n 1 weblog-sample.log | nc localhost 9900
```

得到输出类似如下：

```
{
    "geoip":{
        "longitude":126.97409999999999,
        "ip":"14.49.42.25",
        "country_name":"South Korea",
        "country_code3":"KR",
        "country_code2":"KR",
        "location":{
            "lon":126.97409999999999,
            "lat":37.5112
        },
        "latitude":37.5112,
        "continent_code":"AS",
        "timezone":"Asia/Seoul"
    },
    "request":"/articles/ppp-over-ssh/",
    "@timestamp":"2019-05-12T01:24:44.000Z",
    "port":47450,
    "host":"localhost",
    "timestamp":"12/May/2019:01:24:44 +0000",
    "response":"200",
    "referrer":"\"-\"",
    "ident":"-",
    "useragent":{
        "minor":"6",
        "major":"3",
        "build":"",
        "device":"Other",
        "os_name":"Windows",
        "patch":"b1",
        "name":"Firefox Beta",
        "os":"Windows"
    },
    "@version":"1",
    "verb":"GET",
    "clientip":"14.49.42.25",
    "message":"14.49.42.25 - - [12/May/2019:01:24:44 +0000] \"GET /articles/ppp-over-ssh/ HTTP/1.1\" 200 18586 \"-\" \"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\"",
    "auth":"-",
    "httpversion":"1.1",
    "bytes":"18586",
    "agent":"\"Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.2b1) Gecko/20091014 Firefox/3.6b1 GTB5\""
}
```

# 四、输出

将数据输出到 Elasticsearch：

```
input {
  tcp {
    port => 9900
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
    source => "agent"
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
  }
}
```

读入 weblog-sample.log 的第一行数据：

```
head -n 1 weblog-sample.log | nc localhost 9900
```

打开 Kibana 在 Dev Tools 输入命令：

```
GET logstash/_search
```

可以看到从 Logstash 导入的数据

> 参考

1. [如何安装Elastic栈中的Logstash](https://elasticstack.blog.csdn.net/article/details/99655350)
1. [Logstash Directory Layout](https://www.elastic.co/guide/en/logstash/7.7/dir-layout.html)
1. [Logstash：Logstash 入门教程 （二）](https://elasticstack.blog.csdn.net/article/details/105979677)
1. [Filter plugins](https://www.elastic.co/guide/en/logstash/7.7/filter-plugins.html)
