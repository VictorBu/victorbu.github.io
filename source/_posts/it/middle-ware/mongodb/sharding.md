---
title: MongoDB 分片集群配置
date: 2019-08-07 09:30:00
updated: 2019-08-07 09:30:00
categories: [IT]
tags: [Spring Boot, MongoDB]
---

> 本文测试环境为 CentOS 7 和 MongoDB 最新版 (4.0.12)

使用 root 操作 (实际操作中使用非 root 账户启动报错)

# 零、服务器分配

| 服务器 102  | 服务器 103  | 服务器 104  |
|  ----  | ----  | ----  |
| mongos  | mongos | mongos |
| config server  | config server | config server |
| shard server 1 主节点  | shard server 1 副节点 | 	shard server 1 仲裁 |
| shard server 2 仲裁  | shard server2 主节点 |  shard server 2 副节点 |
| shard server 3 副节点  | shard server 3 仲裁 | shard server 3 主节点 |

端口：

```
mongos：20000
config：21000
shard1：27001
shard2：27002
shard3：27003
```

# 一、MongoDB 安装

**在 3 台服务器分别操作**

## 1.1. 下载

```
cd /usr/local
wget http://downloads.mongodb.org/linux/mongodb-linux-x86_64-rhel70-v4.0-latest.tgz

tar -xzvf mongodb-linux-x86_64-rhel70-v4.0-latest.tgz -C /usr/local/
mv mongodb-linux-x86_64-rhel70-4.0.12-rc0-3-gc57d7cb mongodb
```
## 1.2. 建立文件夹

```
mkdir -p /usr/local/mongodb/conf
mkdir -p /data/mongodb/mongos/log
mkdir -p /data/mongodb/config/data
mkdir -p /data/mongodb/config/log
mkdir -p /data/mongodb/shard1/data
mkdir -p /data/mongodb/shard1/log
mkdir -p /data/mongodb/shard2/data
mkdir -p /data/mongodb/shard2/log
mkdir -p /data/mongodb/shard3/data
mkdir -p /data/mongodb/shard3/log
```

## 1.3. 环境变量

```
vim /etc/profile

```

在文件末尾添加：

```
export MONGODB_HOME=/usr/local/mongodb
export PATH=$MONGODB_HOME/bin:$PATH
```
使配置立即生效：
```
source /etc/profile
```

# 二、config server 配置

**在 3 台服务器分别操作**

```
vim /usr/local/mongodb/conf/config.conf
```

内容：

```
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/config/log/congigsrv.log

# Where and how to store data.
storage:
  dbPath: /data/mongodb/config/data
  journal:
    enabled: true

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /data/mongodb/config/log/configsrv.pid
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 21000
  bindIp: 0.0.0.0

# sharding Options
sharding:
  clusterRole: configsvr
replication:
  replSetName: config
```

启动服务：

```
mongod -f /usr/local/mongodb/conf/config.conf
```

**以下操作在任意一台服务器操作即可**

```
mongo --port 21000
```

```
# config 变量
config = {
    _id : "config",
    members : [
        {_id: 0, host: "192.168.30.102:21000" },
        {_id: 1, host: "192.168.30.103:21000" },
        {_id: 2, host: "192.168.30.104:21000" }
    ]
}

# 初始化
rs.initiate(config)
```

# 三、shard server 配置

**在 3 台服务器分别操作**

```
vim /usr/local/mongodb/conf/shard1.conf
```

内容：

```
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/shard1/log/shard1.log

# Where and how to store data.
storage:
  dbPath: /data/mongodb/shard1/data
  journal:
    enabled: true

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /data/mongodb/shard1/log/shard1.pid
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 27001
  bindIp: 0.0.0.0

# sharding Options
sharding:
  clusterRole: shardsvr
replication:
  replSetName: shard1
```
启动服务：

```
mongod -f /usr/local/mongodb/conf/shard1.conf
```

**以下操作在任意一台服务器操作即可 (实际操作中需要在非裁判服务器操作)**

```
mongo --port 27001
```

```
use admin

# "arbiterOnly":true 代表其为仲裁节点
config = {
    _id: "shard1",
    members: [
        {_id: 0, host: "192.168.30.102:27001"},
        {_id: 1, host: "192.168.30.103:27001"},
        {_id: 2, host: "192.168.30.104:27001", arbiterOnly: true}
    ]
}
# 初始化
rs.initiate(config)
```
**重复上述操作配置 shard2 和 shard3, 注意修改名称、端口和对应的 arbiterOnly**

# 四、mongos 配置

**在 3 台服务器分别操作**

```
vim /usr/local/mongodb/conf/mongos.conf
```

内容：

```
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/mongos/log/mongos.log

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /data/mongodb/mongos/log/mongos.pid
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 20000
  bindIp: 0.0.0.0

# sharding Options
# config 为配置服务器的副本集名字
sharding:
  configDB: config/192.168.30.102:21000,192.168.30.103:21000,192.168.30.104:21000
```

启动服务:

```
mongos -f /usr/local/mongodb/conf/mongos.conf
```

**以下操作在任意一台服务器操作即可**
```
mongo --port 20000
```

```
use admin

# 串联路由服务器与分配副本集
sh.addShard("shard1/192.168.30.102:27001,192.168.30.103:27001,192.168.30.104:27001")
sh.addShard("shard2/192.168.30.102:27002,192.168.30.103:27002,192.168.30.104:27002")
sh.addShard("shard3/192.168.30.102:27003,192.168.30.103:27003,192.168.30.104:27003")
```

# 五、测试

在其中一台 mongos 继续操作：

```
# 指定 test 数据库分片生效
db.runCommand({ enablesharding :"test"})

# 指定数据库里需要分片的集合和片键
db.runCommand({ shardcollection : "test.table1",key : {id: 1}})

# 插入测试数据
for(var i = 1; i <= 100000; i++) db.table1.save({id:i,"test1":"testval1"})
```

# 六、在 Spring Boot 中使用

在其中一台 mongos 继续操作：

```
# 创建账户
db.createUser({
 user: 'test',
 pwd: '123456',
 roles: [{role: "readWrite", db: "test"}]
})
```

Spring Boot 连接字符串：
```
spring:
  data:
    mongodb:
      uri: mongodb://test:123456@192.168.30.102:20000,192.168.30.103:20000,192.168.30.104:20000/test
```

> 参考：

1. [mongodb 3.4 集群搭建：分片+副本集](https://www.cnblogs.com/ityouknow/p/7344005.html)
1. [mongodb 3.4 集群搭建升级版 五台集群](https://www.cnblogs.com/ityouknow/p/7566682.html)
1. [MongoDB Configuration File Options](https://docs.mongodb.com/manual/reference/configuration-options/)




































