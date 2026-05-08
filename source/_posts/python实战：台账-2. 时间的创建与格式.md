---
title: 'python实战: 台账-2. 时间的创建与格式'
date: 2026-04-24 14:00:22
tags: [Python]
categories: [python实战--台账]
---

- 今天学习了如何获取今天的时间, 时间的格式化输出
- `today_md = datetime.date.today().strftime("%m月%d日").lstrip("0").replace("月0", "月")` 
<!-- more -->

## 原版代码

```python
import datetime

today_md = datetime.date.today().strftime("%m月%d日").lstrip("0").replace("月0", "月")

# 功能: 获取今天的日期, 格式化为月-日,自动去掉0, 如01月02日, 变为1月2日
# 逻辑: 先获取今天的日期 格式化成想要的样式
# 易错点: 对lstrip()和replace()的使用
```

## 知识点
~~~python
datetime.date.today()  # 获取今天的日期 如2025-04-05
date.strftime(format)  # 返回一个由显式格式字符串所控制的，代表日期的字符串
str.lstrip([chars])    # 返回原字符串的副本，移除其中的前导字符 strip:除去、剥去
str.replace(old, new[, count])
# 用 new 替换子字符串 old 的所有出现次数，并返回该字符串的副本。如果给定了可选参数 count，则只替换前 count 次出现的字符串。
~~~

