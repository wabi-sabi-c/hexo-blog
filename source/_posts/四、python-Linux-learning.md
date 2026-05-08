---
title: 四、python&Linux_learning
date: 2026-04-30 11:05:00
tags: [Python, Linux]
categories: [自学记录]
---
- 今天继续学习python，默写出上一篇的计算器代码或者熟悉整个框架，for循环练习
- Linux还是复习练习这些命令：`touch、 echo、 cat、 cp、 mv、 rm、 mkdir、 rmdir、 ls、 pwd、 ip addr、 ping、 cd `
<!-- more -->

# python

```python
'''
1. 复盘巩固: 把昨天食堂计算器核心逻辑过一遍
- try-except 作用
- lower() 转小写
'''
# 食堂计算器
def cateen_calculator():
    print("欢迎来到食堂计算器！")
    print("请输入需要计算的两个数字和运算符：")
    print("按'q'退出食堂计算器")

    while True:
        # 先判断是否需要按'q'退出
        print("是否使用食堂计算机？按'q'退出，回车继续使用")
        cleck = input()
        if cleck.lower() == 'q':
            print("欢迎下次再来！")
            break
        try:
            # 获取两个数字和运算符
            num1 = float(input("请输入第一个数字："))
            num2 = float(input("请输入第二个数字："))
            op = input("请输入运算符(加: + 减: - 乘: * 除: / )：")

            # 判断运算符是否输入正确
            if not op in ['+', '-', '*', '/']:
                print("\n请输入正确的运算符(加: + 减: - 乘: * 除: / )\n")
                continue

            # 进行判断是否计算这个计算公式
            print(f"是否计算 {num1} {op} {num2} =  ？ (Y/N)")
            check = input()
            if check.lower() == 'n':
                print("已取消计算！")
                continue

            # 进行计算
            if op == '+':
                result = num1 + num2
            elif op == '-':
                result = num1 - num2
            elif op == '*':
                result = num1 * num2
            elif op == '/':
                # if num2 == 0:
                #     print("除数不能为0！")
                #     continue
                result = num1 / num2

            print(f"计算机结果为：{num1} {op} {num2} = {result}")




        except ValueError:
            print("错误：请输入有效的数字！")
        except ZeroDivisionError:
            print("错误：除数不能为0！")

# 运行程序
if __name__ == '__main__':
    cateen_calculator()
```

```python
'''
2. 练习for循环：
- 创建一个列表，并使用for循环遍历列表中的元素
- 创建一个字典，并使用for循环遍历字典中的元素
- 创建一个元组，并使用for循环遍历元组中的元素
- 创建一个字符串，并使用for循环遍历字符串中的元素
- 创建一个列表，并使用for循环遍历列表中的元素，并打印出索引和元素
- 创建一个字典，并使用for循环遍历字典中的元素，并打印出键和值
'''

def for_group_test():
    print("创建一个列表，并使用for循环遍历列表中的元素")
    list1 = [1, "A", 3, True, 5.0, 'b']
    print(len(list1))
    for i in list1:
        print(f"列表中的元素为：{i} 其类型是 {type(i)}")
    print("创建一个字典，并使用for循环遍历字典中的元素")
    dict1 = {'a': 1, "你好": 2, 'c': 4.5, 'd': 6}
    for i in dict1:
        print(f"字典的键是：{i} 值是：{dict1[i]}")
     
    print("创建一个元组，并使用for循环遍历元组中的元素")
    tuple1 = (1, "A", 3, True, 5.0, 'b')
    for i in tuple1:
        print(f"元组中的元素为：{i}")
    print("创建一个字符串，并使用for循环遍历字符串中的元素")
    string1 = "hello world" 
    print(len(string1))
    for i in string1:
        print(f"字符串中的元素为：{i}")
       
    print("创建一个列表，并使用for循环遍历列表中的元素，并打印出索引和元素")
    list1 = [1, "A", 3, True, 5.0, 'b']
    for i in range(len(list1)):
        print(f"列表中的元素为：{list1[i]} 索引为：{i}")

    print("创建一个字典，并使用for循环遍历字典中的元素，并打印出键和值") 
    dict1 = {'a': 1, "你好": 'b', 'c': 4.5, 'd': True}
    for i in dict1.keys():
        print(f"字典的键是：{i} 值是：{dict1[i]}")
   
        
# 运行程序
if __name__ == '__main__':
    for_group_test()

```
## 踩坑与解决
```python

1. 关于列表打印索引和元素
for i in range(len(list1)):
    print(f"列表中的元素为：{list1[i]} 索引为：{i}")
- for i in list1 -> 只能获取元素，不能获取索引
- 所以先len(list1) -> 获取列表长度
- 再range() -> 生成0到列表长度的数字即索引
- 最后 i 就是索引 列表中的元素就是 list1[i]
- 推荐写法：
    for index, value in enumerate(list1):
        print(f"索引：{index}，元素：{value}")

2. 关于字典打印键和值
- 写法1（你第一次写的）：
    for i in dict1:
        print(f"键：{i} 值：{dict1[i]}")

- 写法2（你最后写的）：
    for i in dict1.keys():
        print(f"键：{i} 值：{dict1[i]}")
- 总结：两个方法的运行结果完全相同, 但写法1是直接遍历字典的所有，
    写法2明确表明遍历字典的键

- 推荐使用写法3：
- 直接同时拿 键+值，不用 dict1[i]
    for key, value in dict1.items():
        print(f"键：{key}，值：{value}")

3. 字典的键、值，支持 数字 / 字符串 / 布尔 / 浮点数 任意类型，混着写完全不报错！
dict1 = {'a': 1, "你好": 'b', 'c': 4.5, 'd': True}

```

# Linux
## 复习Linux命令
~~~bash
# 创建目录
mkdir test
# 查看目录
ls 
ls -l  # 显示详细信息
ls -lh #显示详细信息并显示大小
ls -R  # 递归查看
# 进入目录
cd test
cd .. # 返回上一级目录
cd ~  # 返回根目录
# 删除目录
rmdir test
rm -rf test # 强制递归删除 -i会提示确认
# 创建文件
touch test.txt
# 写入内容
echo "hello world" > test.txt
# 追加在内容的末尾
echo "hello world2" >> test.txt
# 查看文件
cat test.txt
# 删除文件
rm test.txt
rm -rf test.txt #强制递归删除 
# 重命名/移动文件
mv old_name.txt new_name.txt # 重命名（相同类型就是重命名即都是文件、都是目录）
mv old_name.txt dir1 # 移动
mv old_name.txt dir1/new_name.txt # 移动并重命名
# 复制文件
cp old_name.txt new_name.txt
cp old_name.txt dir1
~~~

## 上传文件到Linux服务器
```bash
scp 本地文件路径 Linux用户@LinuxIP地址:/Linux目标目录
scp /Users/Desktop/test.py root@192.168.1.1:/root/test
ls  # 查看结果 -R 递归查看所有目录及文件
```