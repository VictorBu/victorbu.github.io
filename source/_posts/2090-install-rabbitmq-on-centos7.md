---
title: 在 CentOS 7 安装 RabbitMQ
date: 2019-08-09 09:30:00
updated: 2019-08-09 09:30:00
categories: [IT]
tags: [RabbitMQ, Linux]
---

# 一、安装 Erlang

RabbitMQ 是使用 Erlang 开发的，所以需要首先安装 Erlang，本文安装其最新版本

添加 repo 文件：

```
sudo vim /etc/yum.repos.d/rabbitmq_erlang.repo
```

文件内容：

```
[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_erlang-source]
name=rabbitmq_erlang-source
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

安装：

```
sudo yum -y install erlang socat
```

# 二、安装 RabbitMQ

下载 RPM 包：

```
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.17/rabbitmq-server-3.7.17-1.el7.noarch.rpm
```

导入 GPG key：

```
sudo rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```

安装 RabbitMQ：

```
sudo rpm -Uvh rabbitmq-server-3.7.17-1.el7.noarch.rpm
```

启动 RabbitMQ：

```
sudo systemctl start rabbitmq-server
```

查看 RabbitMQ 运行状态

```
sudo systemctl status rabbitmq-server
```

将 RabbitMQ 加入开机自启动：

```
sudo systemctl enable rabbitmq-server
```

# 三、RabbitMQ 配置

放行端口：

```
sudo firewall-cmd --zone=public --permanent --add-port=4369/tcp
sudo firewall-cmd --zone=public --permanent --add-port=25672/tcp
sudo firewall-cmd --zone=public --permanent --add-port=5671-5672/tcp
sudo firewall-cmd --zone=public --permanent --add-port=15672/tcp
sudo firewall-cmd --zone=public --permanent --add-port=61613-61614/tcp
sudo firewall-cmd --zone=public --permanent --add-port=1883/tcp
sudo firewall-cmd --zone=public --permanent --add-port=8883/tcp

sudo firewall-cmd --reload
```

启用 RabbitMQ 网页管理插件

```
sudo rabbitmq-plugins enable rabbitmq_management
```

创建管理员用户并授权：

```
sudo rabbitmqctl add_user admin 你的密码
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

在浏览器访问 http://IP:15672 即可进入到  RabbitMQ 网页管理页面

> 参考：
1. [How to Install RabbitMQ Server on CentOS 7](https://www.howtoforge.com/tutorial/how-to-install-rabbitmq-server-on-centos-7/)
1. [github.com/rabbitmq/erlang-rpm](https://github.com/rabbitmq/erlang-rpm)






























