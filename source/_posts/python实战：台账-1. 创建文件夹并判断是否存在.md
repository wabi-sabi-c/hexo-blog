---
title: python实战：台账--创建文件夹并判断是否存在
date: 2026-04-24 13:05:37
tags: [python, python实战]
categories: [python实战--台账]
---

- 今天学习了python的os模块里创建文件夹和判断文件夹是否存在
- 为什么要判断文件夹是否存在呢？因为重复创建文件夹会报错
- 今天我用来`os.makedirs("dir1", exist_ok=True)`来创建文件夹保证程序不会报错
- 不加`exist_ok=True`会报错
<!-- more -->
## 原版代码  
```python
import os
os.makedirs("dir1", exist_ok=True)

# 功能：使用os模块里的makedirs方法创建文件夹
# 逻辑：如果文件夹不存在，则创建文件夹；如果存在，则不创建，不报错、跳过
# 易错点：
# 1. 不加`exist_ok=True`会直接崩溃报错
# 2. 创建文件夹时，如果文件夹名有特殊字符，会报错
```

## 优化代码  
```python
import os

# 定义文件夹名字(统一管理, 方面修改)
dir_name = "dir1"

# 判断文件夹是否存在
if not os.path.exists(dir_name)
    os.makedirs(dir_name)
    print(f"成功创建文件夹: {dir_name}")
else:
    print(f"已存在文件夹: {dir_name}, 请勿重复创建")

# 这个优化在逻辑更加清晰,有提示信息知道程序在干什么
```

## 关键知识点

1. `os.makedirs()`
```python
# python官方文档：
os.makedirs(path, mode=0o777, exist_ok=False)
# 递归目录创建函数,会自动创建到达最后一级目录所需要的中间目录
> path：创建的目录
> mode：创建目录的权限 默认777(谁都可读, 可写, 可执行) 
> 0o777 中的0o表示八进制,因为python规定:权限必须用八进制表示
    这个参数99% 的情况不用管，系统会自动给，不用改
> exist_ok=False 表示如果目录已经存在，则报错; Ture表示如果目录已经存在，则不报错，不创建
> 所有一般写成 os.makedirs(path, exist_ok=True)
```
2. `os.path.exists()`
```python
# python官方文档：
os.path.exists(path)
# exists 中文意思是 存在
# 功能：检查指定路径的文件/文件夹是否存在
# 逻辑：传入一个路径，返回 True（存在）或 False（不存在）
# 易错点：路径写错、大小写问题、权限问题都会返回 False
```