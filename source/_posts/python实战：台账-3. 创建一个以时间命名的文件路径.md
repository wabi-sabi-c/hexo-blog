---
title: python实战：台账--创建一个以时间命名的文件路径
date: 2026-04-24 14:37:58
tags: [python, python实战]
categories: [python实战--台账]
---

- 今天学习定义一个函数：创建一个以时间命名的文件路径
<!-- more -->

```python
import datetime
import os
def get_excel_file():
    # 功能：自动获取今天日期，并格式化成 2025-12-31 这样的字符串
    # 逻辑：每天生成一个以“今天日期”命名的台账文件，不会重复、不会覆盖
    # 易错点：如果不格式化，日期会带时间，文件名会乱码或报错
    date_str = datetime.date.today().strftime("%Y-%m-%d")

    # 功能：拼接文件路径 → 台账/2025-12-31_食品及食品原辅料采购查验记录.xlsx
    # 逻辑：os.path.join 自动适配 Windows/Linux 斜杠，不会报错
    # 易错点：直接写字符串路径，换系统就报错（Linux \ / 不一样）
    return os.path.join("台账", f"{date_str}_食品及食品原辅料采购查验记录.xlsx")
```
1. 这个函数的功能是：自动生成一个以今天日期命名的文件路径。
2. 它不会覆盖旧的文件、不会重复。
3. 使用`os.path.join(path, *paths)`智能地合并一个或多个路径部分。 返回值将是 path 和所有 *paths 成员的拼接，其中每个非空部分后面都紧跟一个目录分隔符，最后一个除外。
4. 因为没有创建xlsx文件只是生成文件名、文件路径，所有并不需要引入openpyxl 模块。
5. 一定要return，返回值要不然这个函数没有任何东西可以调用

## 实战练习
```python
# 仿写一个生成菜品销售记录.xlsx
```

<details>
     
<summary>点击展开隐藏内容</summary>

```python
import datetime
import os
def get_excel_file():
    date_str = datetime.date.today().strftime("%Y-%m-%d")
    file1 = os.path.join("Documents", f"{date_str}菜品销售记录.xlsx")
    return file1

# 测试调用
if __name__ == "__main__":
    path = get_excel_file()
    print(f"今天的台账文件路径：{path}")

# 使用<details><summary>点击展开隐藏内容</summary>这里是隐藏内容</details> 标签隐藏内容
# 注意：
#      1. <details> 与 <summary>点击展开隐藏内容</summary> 必须空一行
#      2. <summary>点击展开隐藏内容</summary> 与 ```python即内容 必须空一行   
``` 

</details>





