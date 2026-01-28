---
title: 在 Linux 使用 ngrok 实现内网穿透
date: 2019-02-19 14:00:00
updated: 2019-02-19 14:00:00
categories: [IT]
tags: [Linux, ngrok]
---

> 本文在 64 位 CentOS 6.7 测试通过

# 注册账号

访问 [ngrok](https://ngrok.com/) 注册账号

# 下载 ngrok

```
wget -P /home/tmp/download https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
```

"/home/tmp/download" 为文件保存目录，如果没有安装 wget 需要先安装 wget

# 解压 ngrok

```
unzip ngrok-stable-linux-amd64.zip
```

# 配置 authtoken

## 查看 token：

![](https://oss.x8y.cc/blog-img/2017/authtoken.png)

## 执行命令

```
./ngrok authtoken YourToken
```

# 启动

```
./ngrok http 端口
```

上面的启动方法退出终端后，ngrok 也会停止运行，所以不太适合。比较好的方式是执行下面命令启动：

```
nohup ./ngrok http 端口 > /dev/null &
```
+ nohup: nohup 是 no hang up 的缩写，该命令可以在退出登录或关闭终端后继续运行进程
+ \> /dev/null: /dev/null 表示空设备，> /dev/null 即不记录日志
+ &: 指在后台运行

查看外网访问地址：

```
curl http://127.0.0.1:4040/api/tunnels
```


> 参考：

1. [ngrok get started](https://dashboard.ngrok.com/get-started)
1. [ngrok running in background](https://stackoverflow.com/questions/27162552/ngrok-running-in-background)