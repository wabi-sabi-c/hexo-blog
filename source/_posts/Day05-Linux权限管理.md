---
title: Day05 Linux权限管理
date: 2026-04-23 15:39:56
tags: [Linux]
categories: [自学记录]
---

# Day05｜Linux 用户、组与权限管理（核心必学）

今天是 180 天运维自学计划第 5 天，学习 Linux **安全核心：用户、组、权限**。
这是面试高频、实操必用、必须吃透的知识点。
<!-- more -->
## 一、Linux 用户分类
- **root（UID=0）**：超级管理员，最高权限
- **系统用户（UID=1~999）**：运行服务专用
- **普通用户（UID=1000+）**：日常操作使用

## 二、用户管理常用命令
### 1. 创建用户
`useradd zhangsan`
### 2. 设置 / 修改密码
`passwd zhangsan`
### 3. 切换用户
`su zhangsan`
### 4. 删除用户
`userdel -r zhangsan`  # -r 同时删除用户目录
### 5. 查看当前用户
`whoami zhangsan`

## 三、用户组管理
### 1. 创建用户组
`groupadd groupname`
### 2. 添加用户到用户组
`usermod -a -G groupname username`
### 3. 查看用户组信息
`cat /etc/group`

## 四、权限管理
### 1. 文件权限
```bash
执行 ll 会看到一行：
-rw-r--r--. 1 root root 12 Apr 23 10:00 test.txt
```
1. `-rw-r--r--` 权限部分（共 10 位）
- 第 1 位：文件类型
   -   `-` 普通文件
   -   `d` 文件夹
   -   `l` 快捷方式
- 第 2~4 位：所有者权限（user）
   -   `rw-` 可读、可写、不可执行
- 第 5~7 位：所属组权限（group）
   -   `r--` 只读
- 第 8~10 位：其他用户权限（others）
   -   `r--` 只读

```bash
1 root root 12 Apr 23 10:00 test.txt
```
- `1` 链接数(不用管)
- `root` 文件所属用户
- `root` 文件所属组
- `12` 文件大小12字节
- `Apr 23 10:00` 文件最后修改时间
- `test.txt` 文件名

**总结：**
只有 root 能读能改这个test.txt文件，别人只能看不能改

### 2. 权限修改

** 在[Day02-Linux-文件与目录常用命令-必背](Day02-Linux-文件与目录常用命令-必背.md)中解释过**
修改权限 `chmod`
修改所有者与所属组 `chown`


## 五、实战练习
```bash
useradd lisi
passwd lisi

touch test.txt
ll test.txt

chmod 755 test.txt
chown lisi test.txt

ll test.txt
```

## 六、Day05 合格标准
1. 会创建 / 删除 / 切换用户
2. 能看懂 `ll` 那一行权限
3. 会用 `chmod 644 / 755`
4. 会用 `chown` 改归属
5. 能独立完成实战练习


# 今天吃透权限，后面部署服务、处理报错都会顺畅很多