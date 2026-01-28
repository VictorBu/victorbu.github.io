---
title: Spring Boot 使用 XXL-JOB
date: 2020-04-21 11:00:00
updated: 2020-04-21 11:00:00
categories: [IT]
tags: [Spring Boot, XXL-JOB]
---

# 一、配置部署调度中心

## 1.1 下载源码

https://github.com/xuxueli/xxl-job

## 1.2 数据库初始化

执行 /xxl-job/doc/db/tables_xxl_job.sql

## 1.3 修改配置

/xxl-job/xxl-job-admin/src/main/resources/application.properties

```
server.port=8080

spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root_pwd

xxl.job.accessToken=
```

上面几个配置根据需求修改，其他的可以不用修改

执行器默认端口是 9999，可以通过 xxl.job.executor.port 修改

## 1.4 部署

编译代码，启动调度中心

```
java -jar xxl-job-admin-2.2.1-SNAPSHOT.jar
```

调度中心访问地址：http://localhost:8080/xxl-job-admin , 默认登录账号：admin/123456

# 二、使用

## 2.1 添加依赖包

```
<dependency>
	<groupId>com.xuxueli</groupId>
	<artifactId>xxl-job-core</artifactId>
	<version>2.2.0</version>
</dependency>
```

## 2.2 添加配置

复制 /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/java/com/xxl/job/executor/core/config/XxlJobConfig.java 到项目

## 2.3 修改配置

```
xxl:
  job:
    admin:
      # 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册
      addresses: http://127.0.0.1:8080/xxl-job-admin
      # 执行器通讯TOKEN [选填]：非空时启用
    accessToken:

    executor:
      # 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
      appname: xxl-job-executor-sample
      # 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
      address:
      # 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
      ip:
      # 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
      port: 9999
      # 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
      logpath: /data/applogs/xxl-job/jobhandler
      # 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
      logretentiondays: 30
```

## 2.4 添加测试任务

1. 在Spring Bean实例中，开发Job方法，方式格式要求为 "public ReturnT<String> execute(String param)"
2. 为Job方法添加注解 "@XxlJob(value="自定义jobhandler名称", init = "JobHandler初始化方法", destroy = "JobHandler销毁方法")"，注解value值对应的是调度中心新建任务的JobHandler属性的值。
3. 执行日志：需要通过 "XxlJobLogger.log" 打印执行日志；

可以拷贝 /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/java/com/xxl/job/executor/service/jobhandler/SampleXxlJob.java 中的代码

## 2.5 调度中心添加新任务

http://localhost:8080/xxl-job-admin/jobinfo ，添加成功后即可看到效果


[ XXL开源社区](https://www.xuxueli.com/xxl-job/)