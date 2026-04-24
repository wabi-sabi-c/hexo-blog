---
title: python实战：台账--初始化Excel表格
date: 2026-04-24 15:25:07
tags: [python, python实战]
categories: [python实战--台账]
---

- 今天学习了使用python操作excel建立台账表格
- 创建一个excel表格，并添加表头
<!-- more -->

## 原版代码
```python
import os
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Alignment, Border, Side, Font


def init_excel():
    excel_file = get_excel_file()
    os.makedirs(os.path.dirname(excel_file), exist_ok=True)
    
    if os.path.exists(excel_file):
        wb = load_workbook(excel_file)
        ws = wb.active
    else:
        wb = Workbook()
        ws = wb.active
        ws.title = "食品及食品原辅料采购查验记录"

        ws.merge_cells('A1:P1')
        ws['A1'] = '食品及食品原辅料采购查验记录'
        ws['A1'].font = Font(bold=True, size=16)
        ws['A1'].alignment = Alignment(horizontal='center', vertical='center')
        ws.row_dimensions[1].height = 30

        ws.merge_cells('A2:A4')
        ws['A2'] = '序号'
        ws.merge_cells('B2:B4')
        ws['B2'] = '产品名称'
        ws.merge_cells('C2:C4')
        ws['C2'] = '规格'
        ws.merge_cells('D2:D4')
        ws['D2'] = '数量'
        ws.merge_cells('E2:E4')
        ws['E2'] = '进货时间'
        ws.merge_cells('F2:F4')
        ws['F2'] = '生产日期或批号'
        ws.merge_cells('G2:G4')
        ws['G2'] = '保质期'
        ws.merge_cells('H2:H4')
        ws['H2'] = '供货商名称'
        ws.merge_cells('I2:I4')
        ws['I2'] = '联系方式'

        ws.merge_cells('J2:N2')
        ws['J2'] = '索证索票情况（有√/无×）'
        ws['J3'] = '营业执照'
        ws.merge_cells('K3:L3')
        ws['K3'] = '许可证'
        ws['M3'] = '产品合格证明'
        ws['N3'] = '检疫证明'
        ws['K4'] = '生产'
        ws['L4'] = '经营'

        ws.merge_cells('O2:O4')
        ws['O2'] = '验收人签字（双人）'
        ws.merge_cells('P2:P4')
        ws['P2'] = '备注'

        thin = Border(left=Side('thin'),right=Side('thin'),top=Side('thin'),bottom=Side('thin'))
        align = Alignment(horizontal='center',vertical='center',wrap_text=True)
        for row in ws.iter_rows(min_row=1, max_row=4):
            for cell in row:
                cell.border = thin
                cell.alignment = align

        widths = {'A':8,'B':18,'C':8,'D':8,'E':12,'F':16,'G':12,'H':30,
                   'I':15,'J':10,'K':10,'L':10,'M':12,'N':12,'O':15,'P':10}
        for col, w in widths.items():
            ws.column_dimensions[col].width = w
        wb.save(excel_file)
    return wb, ws, excel_file

```
## 逐一分析原版代码中的方法
```python
# 第一个
# 创建目录当文件不存在时创建，存在时不创建不报错
os.makedirs(dir_name, exist_ok=True)

# 第二个
# 获取文件目录
os.path.dirname(excel_file)

# 第三个
# 判断文件是否存在
os.path.exists(excel_file)

# 第四个
# 加载excel文件
wb = load_workbook(excel_file)

# 第五个
# 获取当前工作表
ws = wb.active

# 第六个
# 创建工作表
wb = Workbook()

# 第七个
# 单元格合并操作：合并表格第一行的A-P列 
# # 合并取消 对象.unmerge_cells('A1:P1') 
ws.merge_cells('A1:P1')

# 第八个
# 设置单元格内容 定位到 Excel 表格里的 A1 单元格
ws['A1'] = '食品及食品原辅料采购查验记录'

# 第九个
# 设置字体样式 字体加粗、字体大小16号
ws['A1'].font = Font(bold=True, size=16)

# 第十个
# 设置单元格居中 水平居中、垂直居中
# .alignment：是单元格的 “对齐方式” 属性
# horizontal：是单元格的 “水平对齐方式” 属性  center(居中) left（左对齐）right（右对齐）
# vertical  ：是单元格的 “垂直对齐方式” 属性  center(居中) top（上对齐）bottom（下对齐）
ws['A1'].alignment = Alignment(horizontal='center', vertical='center')

# 第十一个
# 设置行高（单位为磅）
# .row_dimensions：工作表中所有行的维度管理对象
# [1]：指定第 1 行（行号从 1 开始）
ws.row_dimensions[1].height = 30

# 第十二个
# 单元格合并操作：合并表格A2-A4行即A列的第二行到第四行
# 这样写也行：
# ws.merge_cells(start_row=2, start_column=1, end_row=4, end_column=1)
ws.merge_cells('A2:A4')
    ws['A2'] = '序号'

# 第十三个
# 合并第二行的J列-N列
ws.merge_cells('J2:N2')
ws['J2'] = '索证索票情况（有√/无×）'
ws['J3'] = '营业执照'
ws.merge_cells('K3:L3')
ws['K3'] = '许可证'
ws['M3'] = '产品合格证明'
ws['N3'] = '检疫证明'
ws['K4'] = '生产'
ws['L4'] = '经营'

# 第十四个
# 设置单元格边框 边框样式 给单元格设置四周都为细边框
# Border：openpyxl.styles 中用于定义单元格边框的类
# left/right/top/bottom：分别指定单元格的左、右、上、下边框
# Side('thin')：指定边框的线型为 thin（细线），还支持 medium（中等线）、thick（粗线）、dashed（虚线）等
# 定义好thin后可以直接使用如ws['A1'].border = thin
thin = Border(left=Side('thin'),right=Side('thin'),top=Side('thin'),bottom=Side('thin'))

# 第十五个
# for循环设置单元格边框
# min_row=1：遍历的起始行号（从第 1 行开始)
# max_row=4：遍历的结束行号（到第 4 行结束)
# 遍历第1-4行，A-C列的单元格
# ws.iter_rows(min_row=1, max_row=4, min_col=1, max_col=3)
# 该方法会返回一个生成器，每次迭代会返回一整行的所有单元格对象
for row in ws.iter_rows(min_row=1, max_row=4):
    for cell in row:
        cell.border = thin
        cell.alignment = align

# 第十六个
# 设置单元格宽度 单位为字符 
widths = {'A':8,'B':18,'C':8,'D':8,'E':12,'F':16,'G':12,'H':30,
           'I':15,'J':10,'K':10,'L':10,'M':12,'N':12,'O':15,'P':10}

# 第十七个
# 遍历widths字典
# col：字典的键
# w：字典的值
# 将工作表 ws 中列 col 的宽度设置为 w（单位为字符数）
for col, w in widths.items():
    ws.column_dimensions[col].width = w

# 第十八个
# 保存文件
wb.save(excel_file)

```

## 总结
1. 学会上面的方法，你就可以熟练生成各式表头的Excel文件了
2. 当然的，这个方法只是最基础的，你可以根据自己需求进行扩展
