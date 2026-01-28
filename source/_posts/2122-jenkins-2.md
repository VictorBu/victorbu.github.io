---
title: Jenkins 之权限 (Role-based Authorization Strategy)
date: 2021-03-31 11:00:00
updated: 2021-03-31 11:00:00
categories: [IT]
tags: [Jenkins]
---

# 一、配置

## 1.1. 安装插件

安装 Role-based Authorization Strategy 插件

## 1.2. 设置授权策略

“系统管理”--“全局安全配置”：

![](https://oss.x8y.cc/blog-img/2122/2/setting.png)

## 1.3. 添加用户

“系统管理”--“管理用户”


# 二、设置所有 item 权限

Global role 为全局权限，比如设置对所有任务均有执行权限，则按照下面设置：

## 2.1. 角色授权

“系统管理”-- “Manage and Assign Roles”--“Manage Roles”：

![](https://oss.x8y.cc/blog-img/2122/2/global-role.png)

## 2.2. 分配角色（用户绑定角色）

“系统管理”-- “Manage and Assign Roles”--“Assign Roles”：

![](https://oss.x8y.cc/blog-img/2122/2/global-role-assign.png)

注：需要勾选“全部” 的 Read 权限

# 三、设置部分 item 权限

Item role 为任务权限，比如设置对所有任务名包含 "demo" 的任务均有执行权限，则按照下面设置：

## 3.1. 角色授权

“系统管理”-- “Manage and Assign Roles”--“Manage Roles”：

![](https://oss.x8y.cc/blog-img/2122/2/item-role.png)

## 3.2. 分配角色（用户绑定角色）

“系统管理”-- “Manage and Assign Roles”--“Assign Roles”：

![](https://oss.x8y.cc/blog-img/2122/2/item-role-assign.png)

注：同样需要设置 Global role 的权限：勾选“全部” 的 Read 权限

