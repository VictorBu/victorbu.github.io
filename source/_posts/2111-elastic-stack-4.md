---
title: 浅尝 Elastic Stack (四) Logstash + Beats 读取 Spring Boot 日志
date: 2020-06-04 21:00:00
updated: 2020-06-04 21:00:00
categories: [IT]
tags: [Elastic, Logstash, Beats, Spring Boot]
---

# 一、Spring Boot 日志配置

采用 Spring Boot 默认的 Logback：

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration  scan="true" scanPeriod="10 seconds">

    <contextName>logback</contextName>
    <property name="LOG_PATTERN" value="%d{ISO8601} %-5level [%thread] %logger - %msg%n" />
    <property name="FILE_PATH" value="/project/logs" />

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">

        <file>${FILE_PATH}/demo.log</file>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>

        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">

            <fileNamePattern>${FILE_PATH}/demo.%d{yyyy-MM-dd}.%i.log</fileNamePattern>

            <maxHistory>30</maxHistory>

            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>

            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>


    </appender>
	
    <root level="INFO">
        <appender-ref ref="console" />
        <appender-ref ref="rollingFile" />
    </root>
</configuration>
```

输出的日志格式如下：

```
2020-06-02 15:05:14,903 INFO  [http-nio-18000-exec-1] com.example.demo.TestController - test log
```

# 二、FileBeat 配置文件

```
filebeat.inputs:

- type: log
  enabled: true
  paths:
    - /project/logs/demo.log
  multiline.pattern: ^(\d{4}|\d{2})\-(\d{2}|[a-zA-Z]{3})\-(\d{2}|\d{4})
  multiline.negate: true
  multiline.match: after

output.logstash:
  hosts: ["localhost:9909"]
```

# 三、Logstash 配置文件

```
input {  
  beats {
    port => "9909"
  }
}

filter {
  grok {
    match => ["message", "%{TIMESTAMP_ISO8601:timestamp}%{SPACE}%{LOGLEVEL:level}%{SPACE}\[%{NOTSPACE:thread}\]%{SPACE}%{NOTSPACE:logger}%{SPACE}-%{SPACE}%{JAVALOGMESSAGE:msg}"]
  }

  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss,SSS"]
  }
}
 
output {
  file {
    path => "/project/logs/9909.1.log"
    }
	
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "spring-boot-log-demo-%{+yyyy.MM.dd}"
  }
}
```

# 四、使用

启动 FileBeat，看到类似如下输出：

```
{
    "@timestamp":"2020-06-02T07:05:14.903Z",
    "level":"INFO",
    "thread":"http-nio-18000-exec-1",
    "host":{
        "name":"localhost.localdomain"
    },
    "timestamp":"2020-06-02 15:05:14,903",
    "logger":"com.example.demo.TestController",
    "log":{
        "offset":78483,
        "file":{
            "path":"/project/logs/demo.log"
        }
    },
    "tags":[
        "beats_input_codec_plain_applied"],
    "@version":"1",
    "ecs":{
        "version":"1.5.0"
    },
    "message":"2020-06-02 15:05:14,903 INFO  [http-nio-18000-exec-1] com.example.demo.TestController - test log",
    "input":{
        "type":"log"
    },
    "agent":{
        "ephemeral_id":"64285633-8b79-4ae6-be6a-02fe5da31866",
        "hostname":"localhost.localdomain",
        "id":"c441fd1d-e158-4712-b3cf-1d2413920532",
        "version":"7.7.0",
        "type":"filebeat"
    },
    "msg":"test log"
}
```

上面的 log、message 等信息我们不需要，可以在配置中去掉，修改 Logstash 配置：

```
input {  
  beats {
    port => "9909"
  }
}

filter {
  grok {
    match => ["message", "%{TIMESTAMP_ISO8601:timestamp}%{SPACE}%{LOGLEVEL:level}%{SPACE}\[%{NOTSPACE:thread}\]%{SPACE}%{NOTSPACE:logger}%{SPACE}-%{SPACE}%{JAVALOGMESSAGE:msg}"]
  }

  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss,SSS"]
  }

  mutate {
    remove_field => "log"
	remove_field => "message"
  }
}
 
output {
  file {
    path => "/project/logs/9909.1.log"
    }
	
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "spring-boot-log-demo-%{+yyyy.MM.dd}"
  }
}
```

修改后效果：

```
{
    "@timestamp":"2020-06-02T07:40:46.112Z",
    "level":"INFO",
    "thread":"http-nio-18000-exec-9",
    "host":{
        "name":"localhost.localdomain"
    },
    "timestamp":"2020-06-02 15:40:46,112",
    "logger":"com.example.demo.TestController",
    "tags":[
        "beats_input_codec_plain_applied"],
    "@version":"1",
    "ecs":{
        "version":"1.5.0"
    },
    "input":{
        "type":"log"
    },
    "agent":{
        "ephemeral_id":"64285633-8b79-4ae6-be6a-02fe5da31866",
        "version":"7.7.0",
        "type":"filebeat",
        "hostname":"localhost.localdomain",
        "id":"c441fd1d-e158-4712-b3cf-1d2413920532"
    },
    "msg":"test log"
}
```

> 实际使用中发现 Logstash 启动的服务端口不能超过两个，不然会接收不到上传的信息

> 参考

1. [filebeat+redis+ELK收集Springboot的Logback日志](https://www.jianshu.com/p/a55dee5aecd2)