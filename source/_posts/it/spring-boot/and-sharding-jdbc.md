---
title: Spring Boot + Sharding-JDBC 读写分离
date: 2019-07-26 12:30:00
updated: 2019-07-26 12:30:00
categories: [IT]
tags: [Spring Boot, Sharding-JDBC, MySQL]
---

> 本文使用 Sharding-JDBC 实现读写分离，基于 CentOS 7 + MySQL 5.7

# 一、MySQL 安装及配置

## 1.1 安装

依次执行命令：

```
sudo wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

sudo yum -y install mysql57-community-release-el7-10.noarch.rpm

sudo yum -y install mysql-community-server

sudo yum -y remove mysql57-community-release-el7-10.noarch
```

启动：

```
sudo systemctl start  mysqld
```

## 1.2 修改密码

查看默认密码：

```
grep "password" /var/log/mysqld.log
```

进入数据库：

```
mysql -uroot -p
```

修改密码：

```
alter user 'root'@'localhost' identified by 'NEW PASSWORD';
```

远程访问：

```
use mysql;
grant all privileges on *.* TO 'root'@'%' identified by 'PASSWORD';
flush privileges;
```

## 1.3 主从配置

本文一主 (192.168.30.101) 两从 (192.168.30.102, 192.168.30.103)

### 1.3.1 主库

```
sudo vim /etc/my.cnf
```

```
# server-id 给数据库服务的唯一标识
server-id=101
# log-bin 设置此参数表示启用 binlog 功能，并指定路径名称
log-bin=/var/lib/mysql/mysql-bin
sync_binlog=0
# 设置日志过期天数
# binlog-ignore-db 表示同步时忽略的数据库
# binlog-do-db 表示需要同步的数据库
expire_logs_days=7
binlog-do-db=test
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
binlog-ignore-db=performance_schema
```

重启数据库，执行 SQL:

```
grant replication slave on *.* to 'root'@'192.168.30.102' identified by 'YOUR PASSWORD';
flush privileges;

grant replication slave on *.* to 'root'@'192.168.30.103' identified by 'YOUR PASSWORD';
flush privileges;
```

重启数据库，执行 SQL:

```
show master status;
```

记录下 File 和 Position


### 1.3.2 从库配置

以 192.168.30.102 为例：

```
log-bin=mysql-bin
server-id=102
binlog-ignore-db=information_schema
binlog-ignore-db=sys
binlog-ignore-db=mysql
replicate-do-db=test
replicate-ignore-db=mysql
log-slave-updates
slave-skip-errors=all
slave-net-timeout=60
```

重启数据库，执行 SQL:

```
stop slave;
change master to master_host='192.168.30.101',master_user='root',master_password='YOUR PASSWORD',master_log_file='mysql-bin.000002', master_log_pos=154;
start slave;
```

其中 master_log_file 和 master_log_pos 分别为上步记录主库的 File 和 Position


# 二、使用

## 2.1 pom.xml

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.18</version>
</dependency>
<dependency>
    <groupId>io.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
```

## 2.2. application.yml

```
spring:
  main:
    allow-bean-definition-overriding: true

mybatis:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath:mapping/*.xml

sharding:
  jdbc:
    datasource:
      names: db-master-1,db-slave-1,db-slave-2

      db-master-1:
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.30.101:3306/test
        username: root
        password: root
        maxPoolSize: 20

      db-slave-1:
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.30.102:3306/test
        username: root
        password: root
        maxPoolSize: 20

      db-slave-2:
        type: com.alibaba.druid.pool.DruidDataSource
        driverClassName: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://192.168.30.103:3306/test
        username: root
        password: root
        maxPoolSize: 20

    config:
      masterslave:
        load-balance-algorithm-type: round_robin # random 随机, round_robin 轮询
        name: db1s2
        master-data-source-name: db-master-1
        slave-data-source-names: db-slave-1,db-slave-2

    props:
      sql:
        show: true
```

完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-2-sharding-jdbc)



> 参考：

1. [CentOS7 yum方式安装MySQL5.7](https://www.cnblogs.com/luohanguo/p/9045391.html)
1. [Sharding-JDBC教程：Mysql数据库主从搭建](https://www.fangzhipeng.com/db/2019/06/25/mysql-install-ms.html)
1. [Sharding-JDBC教程：Spring Boot整合Sharding-JDBC实现读写分离](https://www.fangzhipeng.com/db/2019/06/26/shardingjdbc-master-slave.html)
