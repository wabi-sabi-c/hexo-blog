---
title: python实战：台账-5.添加excel文件内容
date: 2026-04-27 12:58:50
tags: [python, python实战] 
categories: [python实战--台账]
---

- 今天学习了往excel文件中添加数据

<!-- more -->

## 原版代码
```python
# 作用：使用openpyxl模块的Workbook类
# 原因：要获取excel文件，并添加数据
# 易错：不导入openpyxl库无法使用excel_mame.cell()方法, 会报错
from openpyxl import Workbook
# 作用：封装一个函数
# 原因：使用封装，方便调用，模块化，获取最后有效行
# 易错：函数名不能与模块名相同，且函数名需要简单易读
def get_last_valid_row(ws):
    # 作用：range()函数：range(start, stop, 步长) 倒序遍历从最后一行开始，到第四行之前结束，不包括第四行，步长：依次减1
    # 原因：需要倒序遍历，一般前几行都是表头、标题等固定信息
    # 易错：range()函数的参数必须是整数，否则会报错
    for r in range(ws.max_row, 4, -1):
        # 作用：判断当前行是否为空，即第r行的第一列（A列）的值
        # 原因：判断是否有内容
        # 易错：cell()方法参数必须是整数，否则会报错
        if ws.cell(r, 1).value is not None:
            # 作用：返回当前行
            # 原因：有值，则返回当前行
            # 易错：None is not None = False，不执行return r
            return r
    # 作用：返回默认行
    # 原因：没有找到有内容的行，则返回默认行
    # 易错：因为代码的设计逻辑是：第 1~4 行是固定的表头 / 标题行，默认认为这几行一定有效
    return 4
```    

```python
from openpyxl import Workbook
from openpyxl.styles import Border, Side, Alignment
import tkinter as tk
from tkinter import messagebox
def add_row():
    name = entry_name.get().strip()
    spec = entry_spec.get().strip()
    qty = entry_qty.get().strip()
    # 作用：将中文句号替换为英文小数点
    # 原因：输入小数时在中文输入法中是句号，来回切换很麻烦
    qty = qty.replace("。", ".")
    # 作用：判断输入是否为空
    # 原因：如果任意值为空，not None = True 执行下面语句
    # 易错：当数量就是0时，就会无法正常填写，因为数字0 是False，not 0 = True 执行下面语句，所以需要数量为0有效时，or qty is None：不执行下面语句
    if not name or not spec or not qty:
        messagebox.showerror("提示","商品、规格、数量不能为空")
        return
    
    try:
        float(qty)
    except:
        messagebox.showerror("提示", "数量必须是数字（支持小数）")
        return

    try:
        wb, ws, file = init_excel()
        last_row = get_last_valid_row(ws)
        new_row = last_row + 1

        last_num = ws.cell(last_row, 1).value
        # 作用：isinstance(对象, 类型/类型元组) 判断last_num是不是int or float 
        # 原因：last_num是最后有效行的第一列中的值即行号
        if isinstance(last_num, (int, float)):
            num = int(last_num) + 1
        else:
            num = 1

        ws[f'A{new_row}'] = num
        ws[f'B{new_row}'] = name
        ws[f'C{new_row}'] = spec
        ws[f'D{new_row}'] = qty
        ws[f'E{new_row}'] = today_md
        ws[f'H{new_row}'] = FIX_SUPPLIER
        ws[f'I{new_row}'] = FIX_PHONE
        ws[f'J{new_row}'] = FIX_CHECK
        ws[f'K{new_row}'] = FIX_CHECK
        ws[f'L{new_row}'] = FIX_CHECK
        ws[f'M{new_row}'] = FIX_CHECK

        thin = Border(left=Side('thin'),right=Side('thin'),top=Side('thin'),bottom=Side('thin'))
        align = Alignment(horizontal='center',vertical='center',wrap_text=True)
        for col in range(1,17):
            c = ws.cell(row=new_row, column=col)
            c.border = thin
            c.alignment = align

        wb.save(file)

        # 清空输入框
        entry_name.delete(0, tk.END)
        entry_spec.delete(0, tk.END)
        entry_qty.delete(0, tk.END)
        # 光标自动定位到name输入框，方便输入下一个商品
        entry_name.focus()

    except PermissionError:
        messagebox.showerror("错误", "台账文件被占用，请先关闭Excel！")
    except Exception as e:
        messagebox.showerror("错误", f"录入失败：{str(e)}")
```

## 优化代码
~~~python
def get_last_valid_row(ws,header_row=4):
    for r in range(ws.max_row, header_row, -1):
        for cell in ws[r]:
        # 作用：判断每一行的每一列是否有内容
        # 原因：万一是第一列没有值但其他列有值
        # 易错：同时处理None值和空白字符这两句要同时出现，str(None).strip() = "None" 是有值的所有要先检测None值，再检测空白字符
        if cell.value is not None and str(cell.value).strip() != "":
            return r
    return header_row
~~~

~~~python
# 获取序号（前面你写的逻辑）
    last_num = ws.cell(last_row, 1).value
    # 改成这样是为了避免int(float类型)出错，如int(3.9) = 3,序号使用整数，所以用 last_num.is_integer() 来判断是小数但也是整数如5.0 true ，5.5 false
    if isinstance(last_num, (int, float)) and (isinstance(last_num, int) or last_num.is_integer()) :
        num = int(last_num) + 1
    else:
        num = 1

    # try except 块后
    finally:
        # 确保文件对象被关闭
        if wb is not None:
            wb.close()
        if file is not None:
            file.close()
~~~