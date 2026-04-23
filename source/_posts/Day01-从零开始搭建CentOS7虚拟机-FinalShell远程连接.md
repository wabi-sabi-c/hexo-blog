---
title: Day01 从零开始搭建CentOS7虚拟机 + FinalShell远程连接
date: 2026-04-22 10:36:42
tags: [Linux, CentOS7, VMware, FinalShell]
categories: [自学记录]
---


# Day01｜从零搭建CentOS7虚拟机 + FinalShell远程连接

今天是运维自学 180 天计划的第一天，任务非常明确：
搭建好我们后续学习要用的 Linux 实验环境。
全程跟着做，零基础也能一次成功。
<!-- more -->
## 一、环境准备
- 虚拟机软件：VMware Workstation
- 系统镜像：CentOS-7-x86_64-Minimal.iso（最小化安装版足够）
- 远程工具：FinalShell

## 二、新建虚拟机关键配置（照着填不出错）
1. 安装 → 稍后安装操作系统
2. 选择 Linux → CentOS 64 位
3. 虚拟机名称自定义，位置放非 C 盘
4. 磁盘大小 20G～40G 均可，单文件存储
5. 自定义硬件：
   - 内存：2GB 及以上
   - CPU：2 核
   - 网络适配器：NAT 模式
6. 加载 ISO 镜像文件

## 三、CentOS7 安装要点
- 语言选择：英文（避免后续字符问题）
- 时区：上海
- 磁盘分区：自动分区即可
- 网络一定要打开！否则装好没 IP
- 设置 root 密码，创建普通用户
- 等待安装完成重启

## 四、开机后基础检查
登录 root 用户，执行两条命令：

查看 IP：
```bash
ip addr
ping www.baidu.com # 能连通说明正常
```
使用`ip addr`命令查看 IP 地址，使用`ping`命令测试百度是否正常。
### 4.1 `ip addr`使用后内容解释
```bash
ens33: 大多数CentOS7虚拟机默认网卡名称
一、 inet xxx.xxx.xxx.xxx/24 brd xxx.xxx.xxx.xxx scope global noprefixroute dynamic
  inet：表示一个IP4地址
  xxx.xxx.xxx.xxx：表示 IP 地址
  24：表示子网掩码（及255.255·255.0）
  brd：表示广播地址
  scope：表示地址的作用范围
  global：表示全局地址（可以访问外网/其他网段）
  dynamic：表示IP地址是DHCP动态获取的
二、valid_lft 1762sec preferred_lft 1762sec
  valid_lft：表示地址有效时间，单位为秒
  preferred_lft：表示地址的优先使用剩余时间，单位为秒
三、inet6 xxx:xxx:xxx:xxx:xxx:xxx:xxx:xxx/64 scope link noprefixroute
  inet6：表示一个IP6地址
  xxx:xxx:xxx:xxx:xxx:xxx:xxx:xxx：表示 IP 地址
  64：表示子网掩码（及ffff:ffff:ffff:ffff:ffff:ffff:ffff:0）
  scope：表示地址的作用范围
  link：表示地址是链接地址（即本机地址）（当前链路（同一局域网段内）有效，不能跨网段路由）
  noprefixroute：表示不会为这个网段自动生成路由，默认路由规则
```
## 五、FinalShell 远程连接
1. 新建 SSH 连接
2. 名称随便填
3. 主机填虚拟机里查到的 IP
4. 端口 22
5. 用户 root，密码是你设置的
6. 连接成功，后续操作全部在这里进行

## 六、Day01 必学 6 条Linux基础命令
1. `pwd`：查看当前目录 （print working directory）
2. `ls`：查看当前目录下的文件 （list）
3. `cd`：进入指定目录 （change directory）
4. `mkdir test`：创建test文件夹 （make directory）
5. `rm -rf test`：强制删除test文件夹及以内的所有文件（很危险删除就无法恢复）（remove directory）
6. `man 命令`：获取命令的使用说明

## 七、今日合格标准（自测）
1. 虚拟机成功安装 CentOS7
2. 能获取 IP 并 ping 通外网
3. FinalShell 能正常连接
4. 会使用上面 5 条基础命令



# 满足以上就算 Day01 圆满完成！



  








