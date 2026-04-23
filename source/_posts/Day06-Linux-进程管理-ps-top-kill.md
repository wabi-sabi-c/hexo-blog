---
title: Day06 Linux 进程管理(ps/top/kill)
date: 2026-04-23 16:15:11
tags: [Linux]
categories: [自学记录]
---

# Day06｜Linux 进程管理（ps/top/kill 必学）

今天是 180 天运维自学计划第 6 天，学习 **Linux 进程管理**。
以后排查卡慢、杀服务、看负载，全靠这几个命令。

## 一、什么是进程？
简单说：
**正在运行的程序 = 进程**
每个进程都有一个唯一编号：**PID**
<!-- more -->
## 二、查看进程 ps（最常用）
### 1. 查看当前用户进程
`ps`
### 2. 查看所有进程
`ps -ef`
### 3. 带详细资源显示
`ps aux`
### 4. 过滤某个进程（实战常用）
`ps -ef | grep mysql`
> PID：进程 ID  UID：进程用户 ID  PPID：父进程 ID  CMD：进程名字

## 三、实时监控进程 top
`top`  退出按：`q`
> PID 进程 ID  %CPU CPU 占用  %MEM 内存占用  COMMAND 命令  用来判断：谁把系统搞卡了。
> 
## 四、结束进程 kill
1. 正常杀死（推荐）`kill PID`
2. 强制杀死（杀不死用这个）`kill -9 PID`
```bash
ps -ef | grep nginx
kill -9 12345
```
## 五、查看端口对应的进程
`netstat -ntlp`/ `ss -ntlp`
> 端口号  进程ID  用 kill 干掉

## 六、查看后台程序 jobs
`jobs`
把程序放到后台：命令后加`&`
`sleep 100 &`

## 七、今日实战
```bash
# 1. 查看所有进程
ps -ef

# 2. 实时监控
top

# 3. 找某个进程
ps -ef | grep ssh

# 4. 查看端口
netstat -ntlp

# 5. 测试杀进程
sleep 200 &
ps -ef | grep sleep
kill -9 进程号
```

## 八、Day06 合格标准
1. 会用 `ps -ef`查看进程
2. 会用 `grep` 过滤进程
3. 会用 `top` 看系统负载
4. 会用 `kill` 和 `kill -9`
5. 知道 PID 是什么

# 今天看进程、kill、端口，具备基础运维排查能力,后面部署服务、处理报错都会顺畅很多 
