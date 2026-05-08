---
title: 三、Python&Linux_learning
date: 2026-04-29 15:25:20
tags: [Python, Linux]
categories: [自学记录]
---

- 今天下午学习了使用python写一个计算器函数，结合前两天的学习内容
- python还学习了一下try-except异常处理
- Linux今天目录操作
- 联动python+Linux，python写一个脚本，在Linux上创建目录并生成一个空文件
<!-- more -->

# python
## 任务 1：整合加减乘除，做一个「食堂计算器函数」
### 第一版AI生成版
~~~python
# 初始AI生成版
def calculator(num1, num2, operator):
    """
    一个简单的计算器函数，支持加减乘除。

    参数:
    num1 (int or float): 第一个操作数
    num2 (int or float): 第二个操作数
    operator (str): 运算符 ('+', '-', '*', '/')

    返回:
    int or float: 计算结果
    str: 如果发生错误，返回错误信息
    """
    
    # 输入验证：检查操作数是否为数字
    if not isinstance(num1, (int, float)) or not isinstance(num2, (int, float)):
        return "错误：操作数必须是数字"

    # 输入验证：检查运算符是否合法
    if operator not in ['+', '-', '*', '/']:
        return "错误：不支持的运算符。请使用 '+', '-', '*', '/'"

    # 执行计算
    if operator == '+':
        return num1 + num2
    elif operator == '-':
        return num1 - num2
    elif operator == '*':
        return num1 * num2
    elif operator == '/':
        # 额外验证：除数不能为零
        if num2 == 0:
            return "错误：除数不能为零"
        return num1 / num2

# 测试示例
if __name__ == "__main__":
    print(calculator(10, 5, '+'))   # 输出: 15
    print(calculator(10, 5, '-'))   # 输出: 5
    print(calculator(10, 5, '*'))   # 输出: 50
    print(calculator(10, 5, '/'))   # 输出: 2.0
    print(calculator(10, 0, '/'))   # 输出: 错误：除数不能为零
    print(calculator(10, 'a', '+')) # 输出: 错误：操作数必须是数字
    print(calculator(10, 5, '%'))   # 输出: 错误：不支持的运算符。请使用 '+', '-', '*', '/'

~~~
### 第二版仿写版
```python
'''
1.更改参数，仿写一个计算器(要求独自完成，哪怕错误也硬着头皮写完)
2.使用try-except 异常处理（比 isinstance 更实用）
'''
def canteen_calculator():

    print("欢迎来到食堂计算器！")
    print("请输入两个数字和计算方式（加:  +  减:  -  乘:  *  除:  /  ）：")
    print("如果不需要请按‘ q ’退出。")
    while True:
        try:
            
            num1 = float(input("请输入第一个数字："))
            num2 = float(input("请输入第二个数字："))
            operator = input("请输入运算符：")
            if  num1 or  num2 or  operator == 'q':
                print("感谢使用！")
                break
        except ValueError:
            print("输入错误，请输入有效的数字！")
            continue
        
    print(f"你是否要计算{num1} {operator} {num2} =  ")
    if operator == '+':
        result = num1 + num2
        return result
    elif operator == '-':
        result = num1 - num2
        return result
    elif operator == '*':
        result = num1 * num2
        return result
    elif operator == '/':
        if num2 == 0:
            return "错误：除数不能为零"
        result = num1 / num2
        return result
    else:
        return "错误：不支持的运算符。请使用 '+', '-', '*', '/'"
    

    print(f"结果是：{num1} {operator} {num2} = {result}")


if __name__ == "__main__":
    canteen_calculator()
```
### 第三版AI辅助修改错误
```python
# AI辅助，改正错误
'''
核心 bug1：退出判断逻辑错误
    原代码：if num1 or num2 or operator == 'q':
    这个意思是 “num1 不为空 或者 num2 不为空 或者 operator 是 q”，
    只要输入了数字，条件就成立，直接 break，所以一输入就退出。
核心 bug2：计算代码在 while 循环外面，循环里只做了输入，没计算就退出了。
核心 bug3：输入 q 的逻辑应该在最前面，比如先让用户选是否退出，而不是最后判断。
核心 bug4：return 的位置错误，应该在计算逻辑里，而不是循环外。
核心 bug5：流程错误，应该先判断是否退出，再输入数字，再计算，循环执行。

结构调整：
- 先判断是否输入 q 退出
- 再输入数字和运算符
- 计算逻辑放在循环内
- 修复缩进和逻辑
- 优化交互，让程序正常运行
'''
# 食堂计算器 - 修复完整版
def canteen_calculator():
    print("欢迎来到食堂计算器！")
    print("请输入两个数字和计算方式（加: + 减: - 乘: * 除: /）")
    print("输入 'q' 随时退出程序\n")

    # 无限循环，直到输入q退出
    while True:
        try:
            # 第一步：先判断是否退出
            check = input("输入 q 退出，按回车继续计算：")
            # check.lower() 转换为小写
            if check.lower() == 'q':
                print("感谢使用！")
                break

            # 第二步：输入数字和运算符
            num1 = float(input("请输入第一个数字："))
            num2 = float(input("请输入第二个数字："))
            operator = input("请输入运算符(加: + 减: - 乘: * 除: /)：")

            # 第三步：开始计算（全部放在循环内！）
            print(f"\n是否计算：{num1} {operator} {num2} =   (Y/N)")
            if input().lower() == 'y':
                if operator == '+':
                    result = num1 + num2
                elif operator == '-':
                    result = num1 - num2
                elif operator == '*':
                    result = num1 * num2
                elif operator == '/':
                    if num2 == 0:
                        print("错误：除数不能为零")
                        continue
                    result = num1 / num2
                else:
                    print("错误：不支持的运算符！请输入 加: + 减: - 乘: * 除: / ")
                    continue

                # 输出最终结果
                print(f"计算结果为：{num1}{operator}{num2} = {result}")
            else:
                print("取消计算！,重新开始")
                continue
            

        # 捕获输入非数字的错误
        except ValueError:
            print("❌ 输入错误，请输入有效的数字！\n")

if __name__ == "__main__":
    canteen_calculator()
```
### 第四版继续优化完整版
```python
# 继续优化
'''
bug1：流程顺序反了。错误的运算符也会先弹出 “是否计算？”，
用户选 Y 之后才报错，体验非常不好

结构调整：
-先校验运算符是否有效，再让用户确认是否计算
'''
def canteen_calculator():
    print("欢迎来到食堂计算器！")
    print("请输入两个数字和计算方式（加: + 减: - 乘: * 除: /）")
    print("输入 'q' 随时退出程序\n")

    while True:
        try:
            # 第一步：判断是否退出
            check = input("输入 q 退出，按回车继续计算：")
            if check.lower() == 'q':
                print("感谢使用！")
                break

            # 第二步：输入数字和运算符
            num1 = float(input("请输入第一个数字："))
            num2 = float(input("请输入第二个数字："))
            operator = input("请输入运算符(加: + 减: - 乘: * 除: /)：")

            # 【核心修改1】先校验运算符是否有效（提前拦截错误）
            if operator not in ['+', '-', '*', '/']:
                print("❌ 错误：不支持的运算符！请使用 加: + 减: - 乘: * 除: / 中的一个\n")
                continue

            # 【核心修改2】运算符合法后，再让用户确认是否计算
            print(f"\n确认计算：{num1} {operator} {num2} = ?  (Y/N)")
            confirm = input().lower()
            if confirm == 'n':
                print("取消计算，重新开始\n")
                continue

            # 第三步：用户确认后，再执行计算
            if operator == '+':
                result = num1 + num2
            elif operator == '-':
                result = num1 - num2
            elif operator == '*':
                result = num1 * num2
            elif operator == '/':
                # 除法单独处理除数为0的特殊情况
                if num2 == 0:
                    print("❌ 错误：除数不能为零！\n")
                    continue
                result = num1 / num2

            # 输出最终结果
            print(f"✅ 计算结果：{num1} {operator} {num2} = {result}\n")

        except ValueError:
            print("❌ 输入错误，请输入有效的数字！\n")

if __name__ == "__main__":
    canteen_calculator()
```



## 任务 2：学习 try-except 异常处理（比 isinstance 更实用）
### 何为try-except 异常处理？
1. 异常处理: 就是程序在运行过程中，如果出现了异常（错误），就会停止运行，并给出错误信息。
2. try-except 作用：接住报错 -> 不让程序崩溃 -> 给用户提示 -> 继续运行
### 怎么使用try-except ？
~~~python
try：
    # 可能会报错的语句
except 异常类型:
    # 错误后处理语句
    # 提示信息
#当然可以添加多个except
except 错误类型2:
    # 错误后处理语句
    # 提示信息
~~~
1. try-except 语法 当try-except 运行时，如果 try 块中出现了异常，那么 try 块中的代码就会停止运行，并跳转到 except 块中。
2. try-except 运行时，如果 try 块中没有异常，那么 try 块中的代码就会正常执行，不跳转到 except 块中。
3. except后必须写异常类型，常见的异常类型有：ValueError, TypeError,Exception,ZeroDivisionError等等

# Linux
## 任务 1：学习目录操作命令
```bash
# 1. 创建目录
mkdir canteen  # 创建食堂项目目录
mkdir canteen/code  # 创建代码子目录
mkdir canteen/data  # 创建数据子目录
mkdir canteen/backup  # 创建备份子目录

# 2. 查看目录结构
ls -R  # 递归查看所有子目录

# 3. 删除空目录
rmdir test  # 只能删空目录
```
## 任务 2：掌握 rm -rf 的正确用法（重点！）
```bash
# 1. 先创建几个测试文件和目录
mkdir test_dir
touch test_dir/file1.txt test_dir/file2.txt
# 2. 安全删除非空目录
rm -ri test_dir  # 递归删除目录及其所有内容并提示（缺点：文件多时会一直提示）
# 3. 强制删除非空目录（绝对禁止使用rm -rf 命令删除根目录（/、/*）
rm -rf test_dir
```
## 任务3: 实战：Python + Linux 联动 
- 用 Python 写一个脚本，自动在 Linux 上创建 canteen 目录结构，并生成一个空的 records.xlsx 文件
### AI生成python脚本

<details>

<summary>这个脚本有些复杂, 点击查看</summary>

```python
import os          # 导入操作系统接口模块，用于处理文件和目录路径、创建目录等
import sys         # 导入系统特定参数和函数模块，主要用于获取命令行传入的参数

# python 3.6+
def setup_canteen_structure(base_path="."):
    """
    在指定路径下创建 canteen 目录结构，并生成空的 records.xlsx 文件。
    
    参数:
    base_path (str): 基础路径，默认为当前目录 "."
    """
    # 定义需要创建的目录列表
    dirs = [
        "canteen",            # 根目录
        "canteen/data",       # 数据存放目录
        "canteen/logs",       # 日志存放目录
        "canteen/config"      # 配置存放目录
    ]
    
    # 定义需要创建的空文件列表（包含相对路径）
    files = [
        "canteen/data/records.xlsx",  # Excel 记录文件
        "canteen/config/settings.ini",# 配置文件
        "canteen/logs/app.log"        # 应用日志文件
    ]

    try:
        # 1. 循环创建目录
        for dir_path in dirs:
            # 将基础路径和子目录路径拼接成完整路径
            full_dir_path = os.path.join(base_path, dir_path)
            # 检查目录是否已存在
            if not os.path.exists(full_dir_path):
                # 如果不存在，则创建目录（makedirs 可以递归创建多级目录）
                os.makedirs(full_dir_path)
                print(f"目录已创建: {full_dir_path}")
            else:
                # 如果已存在，打印提示信息
                print(f"目录已存在: {full_dir_path}")

        # 2. 循环创建空文件
        for file_path in files:
            # 将基础路径和文件路径拼接成完整路径
            full_file_path = os.path.join(base_path, file_path)
            # 检查文件是否已存在
            if not os.path.exists(full_file_path):
                # 以写入模式 ('w') 打开文件，如果文件不存在则创建
                with open(full_file_path, 'w') as f:
                    pass  # pass 表示什么都不做，立即关闭文件，从而生成一个空文件
                print(f"文件已创建: {full_file_path}")
            else:
                # 如果文件已存在，打印提示信息
                print(f"文件已存在: {full_file_path}")
                
        # 所有操作成功后，打印完成信息
        print("\nCanteen 目录结构初始化完成！")

    except Exception as e:
        # 捕获任何可能发生的异常（如权限不足、磁盘满等）
        print(f"发生错误: {e}")
        # 非正常退出程序，返回状态码 1
        sys.exit(1)

if __name__ == "__main__":
    # 判断脚本是否被直接运行（而不是被其他模块导入）
    
    # 检查命令行是否有传入参数 (sys.argv[0] 是脚本名, sys.argv[1] 是第一个参数)
    # 如果有参数，则使用用户指定的路径；否则使用当前目录 "."
    target_path = sys.argv[1] if len(sys.argv) > 1 else "."
    
    # 调用主函数，传入目标路径
    setup_canteen_structure(target_path)


# python 2.7
'''
1. 移除 f-strings：
- 原代码：print(f"目录已创建: {full_dir_path}")
- 新代码：print("目录已创建: {}".format(full_dir_path))
- 原因：Python 2.7 不支持 f-strings（这是 Python 3.6+ 的特性）。
使用 .format() 方法兼容 Python 2.7 和 3。

2. Print 语句：
-在 Python 2 中，print 是一个语句（print "hello"），但
在本代码中我们使用了括号 print(...)。这在 Python 2 中也
是合法的（会被视为打印一个元组或表达式），为了保持风格统一和兼容性，
直接使用字符串格式化即可。
- 注意：如果希望完全严格的 Python 2 风格，可以去掉括号，
但保留括号在 Python 2 和 3 中都能运行（只要括号内是单个字符串表达式）。

3. 编码声明：
- 添加了 # -*- coding: utf-8 -*-
防止在 Linux 环境下因默认编码问题导致中文字符串报错。
'''

# -*- coding: utf-8 -*-
import os
import sys

def setup_canteen_structure(base_path="."):
    """
    在指定路径下创建 canteen 目录结构，并生成空的 records.xlsx 文件。
    
    参数:
    base_path (str): 基础路径，默认为当前目录。
    """
    # 定义目录结构
    dirs = [
        "canteen",
        "canteen/data",
        "canteen/logs",
        "canteen/config"
    ]
    
    # 定义需要创建的空文件
    files = [
        "canteen/data/records.xlsx",
        "canteen/config/settings.ini",
        "canteen/logs/app.log"
    ]

    try:
        # 1. 创建目录
        for dir_path in dirs:
            full_dir_path = os.path.join(base_path, dir_path)
            if not os.path.exists(full_dir_path):
                os.makedirs(full_dir_path)
                # Python 2 使用 % 格式化或 .format()，且 print 是语句而非函数（除非导入 from __future__ import print_function）
                print("目录已创建: {}".format(full_dir_path))
            else:
                print("目录已存在: {}".format(full_dir_path))

        # 2. 创建空文件
        for file_path in files:
            full_file_path = os.path.join(base_path, file_path)
            if not os.path.exists(full_file_path):
                # 'w' 模式在 Python 2 中默认写入字节，对于空文件没问题
                with open(full_file_path, 'w') as f:
                    pass  # 创建空文件
                print("文件已创建: {}".format(full_file_path))
            else:
                print("文件已存在: {}".format(full_file_path))
                
        print("\nCanteen 目录结构初始化完成！")

    except Exception as e:
        # Python 2 中异常捕获语法相同，但打印错误信息需注意编码
        print("发生错误: {}".format(str(e)))
        sys.exit(1)

if __name__ == "__main__":
    # sys.argv 在 Python 2 中也是列表
    if len(sys.argv) > 1:
        target_path = sys.argv[1]
    else:
        target_path = "."
    
    setup_canteen_structure(target_path)
```
</details>

### 简单python脚本 
```python
import os

# -*- coding: utf-8 -*-

# 创建主目录
os.mkdir("Canteen")
# 创建子目录
os.mkdir("Canteen/date")
os.mkdir("Canteen/code")
os.mkdir("Canteen/backup")

print("目录创建成功！")
```

### 如何上传到Linux并执行
一、 上传文件到Linux：
- 方法1：使用scp命令
```bash
  # 语法: scp 本地文件路径 用户名@LinuxIP:远程路径
  scp C:\Users\Administrator\Desktop\test.py root@192.168.1.1:/root/test
```
- 方法2：使用 SFTP 工具 使用 FileZilla、WinSCP 等图形化软件，连接到 Linux 服务器，直接将文件拖拽到服务器的主目录
- 方法3：直接在Linux中创建(按`i`进入编辑模式；按`Esc`退出编辑模式；按`:wq`保存并退出)

二、 执行Python文件：
1. 确定python环境
~~~bash
python --version     # 查看python版本
# 或者
python3 --version    # 查看python3版本
~~~

2. 运行Python脚本
```bash
# 直接python目标文件
python test.py
# 或者
python3 test.py
# 有时可能需要sudo 权限 指定路径
sudo python test.py /root/test.py
```

3. 验证结果： 执行完成后，查看目录结构
~~~bash
ls
ls -R 
~~~

### 检测结果：
- 结果显示: 传入简单python脚本并创建了目标目录：1个主目录，3个子目录
![效果展示](/images/LinuxCode2.png)