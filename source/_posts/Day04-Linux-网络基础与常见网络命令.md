---
title: Day04 Linux 网络基础与常见网络命令
date: 2026-04-23 9:16:26
tags: [Linux]
categories: [自学记录]
---

# Day04｜Linux 网络基础与常用网络命令

今天是 180 天运维自学计划第 4 天，我们学习 **Linux 网络**。
不管是搭环境、部署服务、排查问题，网络都是必须掌握的核心技能。
<!-- more -->
## 一、查看 IP 地址
### 1. 查看网卡与IP信息
```bash
ip addr
ifconfig
# 大部分输出的内容再Day01 中已经见过了，这里不再赘述

# 获取 IP 地址
hostname -i         # 获取本机 IP

# 查看网关与路由
ip route            # 获取路由信息   

# 查看系统DNS
cat /etc/resolv.conf
```
### 2. 测试网络连通性
```bash
ping www.baidu.com
ping -c 4 www.baidu.com            # 测试 4 次
ping -c 4 -w 2 www.baidu.com       # 测试 4 次，每次等待 2 秒
ping -c 4 -i 2 www.baidu.com       # 测试 4 次，每次间隔 2 秒
ping -s 1000 www.baidu.com         # 测试 1000 字节
> -c ：指定测试次数  -w ：指定测试时间  -i ：指定测试间隔   -s ：指定测试数据包大小
```

## 二、查看端口情况
### 1.查看连通情况
```bash
telnet www.baidu.com 80         # 测试端口是否连通
curl www.baidu.com              # 测试端口是否连通
```

### 2.查看端口占用情况
```bash
# 1. 安装 net-tools（没有就装一下）
yum install -y net-tools
# 2. 查看所有端口
netstat -ntlp
> -n: 以数字显示 IP 和端口  -t: 显示 TCP 端口  -l: 显示正在监听的端口  -p: 显示进程名
```

## 三、防火墙基础操作
1. 查看防火墙状态 `systemctl status firewalld`
2. 临时关闭防火墙（测试环境用） `systemctl stop firewalld`
3. 禁止开机自启 `systemctl disable firewalld`


## 实战训练
```bash
# 1. 查看自己的IP
ip addr

# 2. 测试网络
ping -c 4 www.baidu.com

# 3. 查看网关
ip route

# 4. 查看端口占用
netstat -ntlp

# 5. 查看DNS
cat /etc/resolv.conf
```

## Day04 合格标准
- 会用 ip addr 查看自己的 IP
- 会用 ping 测试网络
- 会用 netstat -ntlp 查看端口
- 知道如何关闭防火墙
- 能判断自己的虚拟机是否能正常上网


# 以上就是今天关于Linux网络的一些基础，几分钟的事情，洒洒水啦！





