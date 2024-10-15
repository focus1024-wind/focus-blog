---
title: Python操作Excel文件，修改Excel样式（openpyxl）
description: Python操作Excel文件，修改Excel样式（openpyxl）
date: 2023-07-26
slug: python/pref_excel_style
image: 
categories:
    - Python
tags:
    - Python
    - Excel
---

在操作Excel表格时，我们有时需要对Excel表中的内容样式进行修改。当Excel文件过大的情况下修改样式单元格较多，修改麻烦，可采用代码脚本的方式来进行Excel样式的修改

# 安装依赖库`openpyxl`
```shell
pip install openpyxl
```

# `openpyxl`的操作
## 加载文件，获取sheet
### 加载文件`load_workbook`
在Excel中，一般把一个文件称为工作薄。在`openpyxl`中可以通过`load_workbook()`方法来加载一个文件，返回`Workbook`对象。`Workbook`对象会保存Excel表中的所有相关信息。
```python
from openpyxl import load_workbook

load_workbook(file_path)
```

### 获取sheet

在一个Excel文件中会有多个`sheet`表格，所以当操作Excel文件时，需要对多个`sheet`分别处理。在对多个`sheet`表格进行处理时，可以通过先获取表格内部所有的`sheetname`，然后在通过`sheetname`获取对应的`sheet`
- `openpyxl`中可以通过`workbook.get_sheet_names()`方法来获取所需所有`sheetname`列表
	- 在`openpyxl`中`workbook.get_sheet_names()`方法将在后续版本废除，可以通过`workbook.sheetnames`属性来获取所有的`sheetname`
- 获取到对应的`sheetname`名称后，可以通过`workbook.get_sheet_by_name(sheetname)`的方法获取对应的`sheet`，然后对`sheet`中的单元格进行操作
	- 在`openpyxl`中`workbook.get_sheet_by_name(sheetname)`方法将在后续版本废除，可以通过`workbook.[sheetname]`属性来获取对应的`sheet`
```python
sheetnames = workbook.get_sheet_names()

for sheetname in sheetnames:
	sheet = workbook.get_sheet_by_name(sheetname)
	...
```
- `openpyxl`中可以通过`workbook.sheetnames`的方法获取所有的`sheetname`。推荐使用
- `openpyxl`中可以通过`workbook.[sheetname]`的方法根据`sheetname`获取对应的`sheet`。推荐使用
```python
sheetnames = workbook.sheetnames

for sheetname in sheetnames:
	sheet = workbook[sheetname]
	...
```

## 遍历单元格
Excel表格格式作为二维的结构化文件存储格式，对其数据遍历读取修改可以按照行的方式或列的方式进行遍历。
### 迭代遍历
在`openpyxl`中可以通过`sheet.rows`按行获取数据或`sheet.columns`按列获取数据。获取到每一行或者每一列的数据后在对其迭代遍历即可获取到具体的单元格`cell`，可以通过`cell.value`获取单元格的具体值。
**按行遍历：**
```python
for row in sheet.rows:
	for cell in row:
		value = cell.value
		...
```
**按列遍历：**
```python
for col in sheet.columns:
	for cell in col:
		value = cell.value
		...
```

### 索引遍历
通过`sheet.rows`和`sheet.columns`属性可以很好对按行或按列对表格进行迭代遍历，但是有时我们在遍历的过程中希望知道当前遍历对象的索引，迭代遍历不能很好的满足我们的需求。
在`sheet.rows`和`sheet.columns`属性中返回的是一个迭代器，所以不能直接根据索引获取内容，所以需要先将其转换成列表然后获取索引来实现遍历
<font color="red">注意，索引遍历因为将`sheet`的属性转存了，所以在索引遍历中对内容的修改并不会影响到`Workbook`对象，所以索引遍历的优点是仅能获取到对应的索引，并不能对属性进行修改。</font>
**按行索引：**
```python
rows = list(sheet.rows)

for row_index in range(sheet.max_row):
	for col_index in range(len(rows[row_index])):
		value = rows[row_index][col_index].value
		...
```
**按列索引：**
```python
columns = list(sheet.columns)

for col_index in range(sheet.max_column):
	for row_index in range(len(columns[col_index])):
		value = columns[col_index][row_index].value
		...
```

## 单元格行高和列宽的修改
### Excel列号与字母的转换
在Excel中，行号以数字`1`为下标开始索引，列号以字母`A`为下标开始索引。在编程语言中一般以下标`0`为下标开始索引。所以在处理列的时候需要将数字下标转换为相应的字母下标来获取对应的列。`openpyxl`提供列专门的转换工具。
- `openpyxl.utils.get_column_letter(index: int)`：实现数字下标到列号字母的转换
- `openpyxl.utils.column_index_from_string(str_col: str)`：实现列号字母到数字下标到转换

### Excel行高修改
在进行Excel行高的修改时，需要先根据对应的行号获取到对应的行，然后对行高修改。
在`openpyxl`中通过`sheet.row_dimensions[row_number]`获取到对应的行，修改`sheet.row_dimensions[row_number].height`属性来修改行高。
```python
rows = list(sheet.rows)

for row_index in range(sheet.max_row):
	for col_index in range(len(rows[row_index])):
		sheet.row_dimensions[row_index + 1].height = 100
```

### Excel列宽修改
在进行Excel列宽的修改时，需要先根据对应的列号索引获取对应列的字母下标，然后根据字母下标获取对应列，对列宽修改。
在`openpyxl`中通过`sheet.column_dimensions[col_number]`获取到对应的列，修改`sheet.column_dimensions[col_number].width`属性来修改列宽。
```python
columns = list(sheet.columns)

for col_index in range(sheet.max_column):
	for row_index in range(len(columns[col_index])):
		sheet.column_dimensions[get_column_letter(col_index + 1)].width = 100
```

## Excel表格文字对齐属性设置
在对Excel的行高和列宽属性进行修改后，由于文字的对齐设置往往会导致部分单元格中字体的显示效果不好。这时我们可以设置文字的对齐属性来修改文字在单元格中的排布。
在Excel中，对齐属性是针对单元格而言的，所以我们需要获取到对应的单元格而不是行列。对齐属性可以分为水平对齐属性和垂直对齐属性，需要对这两个维度的属性分别进行设置。
- 获取单元格：在`openpyxl`中获取单元格是根据`sheet`按照先列后行的维度进行获取
```python
cell = sheet[f"{get_column_letter(col_number)}{row_number}"]
```
- 对齐属性：在`openpyxl`中对齐属性通过对象`Alignment`进行设置修改
	- 水平方向：`horizontal`属性
		- `left`：左对齐
		- `center`：水平居中
		- `right`：右对齐
		- `justify`：两端对齐
	- 垂直方向：`vertical`属性
		- `top`：顶端对齐
		- `center`：垂直居中
		- `bottom`：底端对齐
```python
from openpyxl.styles import Alignment

alignment = Alignment(horizontal="justify", vertical="center")
```
- 修改对齐属性：通过修改单元格的`alignment`属性来修改对齐属性
```python
from openpyxl.styles import Alignment

alignment = Alignment(horizontal="justify", vertical="center")
sheet[f"{get_column_letter(col_number)}{row_number}"].alignment = alignment
```

## 修改单元格框线
在Excel中，针对表格的框线同样也是针对单元格而言的。`openpyxl`中修改框线通过`Border`对象来设置。由于边框线分别有上下左右四个方向的框线，所以需要分别对四个方向的框线进行设置。在`Boder`对象中通过`Side`属性来设置某一方向上的线条。
- `Border`对象：单元格框线
	- 方向：
		- `top`：上边框线条设置
		- `bottom`：下边框线条设置
		- `left`：左边框线条设置
		- `right`：右边框线条设置
	- 线条属性`Side`对象：
		- `style`：设置线条的属性
			- 可选属性：`dashDot`、`dashDotDot`、`dashed`、`dotted`、`double`、`hair`、`medium`、`mediumDashDot`、`mediumDashDotDot`、`mediumDashed`、`slantDashDot`、`thick`、`thin`
			- 线条的可选属性较多，一般选择常用的`thin`线条即可
		- `color`：设置线条颜色，类型`HEX`格式的颜色属性，默认黑色
- 修改单元格边框：通过修改单元格的`border`属性来修改边框颜色
```python
from openpyxl.styles.borders import Border, Side

thin_border = Border(
	top=Side(style='thin'),
	bottom=Side(style='thin'),
	left=Side(style='thin'),
	right=Side(style='thin')
)
sheet[f"{get_column_letter(col_number)}{row_number}"].border = thin_border
```

## 保存Excel文件
在上文中提到，调用`openpyxl.load_workbook()`方法会返回一个`Workbook`对象，对象中包含了Excel中的相关信息属性，我们在后文中对Excel的修改，本质上都是对`Workbook`对象的属性进行修改，所以在保存是是需要调用`workbook.save(file_path)`即可将修改后的内容写入Excel文件中。

# `openpyxl`实战，Excel样式优化
```python
from openpyxl import load_workbook
from openpyxl.utils import get_column_letter, column_index_from_string
from openpyxl.styles import Alignment
from openpyxl.styles.borders import Border, Side


class ExcelStyle:
    # 边框样式
    thin_border = Border(
        top=Side(style='thin'),
        bottom=Side(style='thin'),
        left=Side(style='thin'),
        right=Side(style='thin'),
    )

    # 对齐属性
    alignment = Alignment(horizontal="justify", vertical="center")

    def _get_max_len_cell_index(self, data):
        """
        获取单元格的行数或列号
        data: 行数据或者列数据
        """
        max_len = 0
        max_len_index = 0
        for cell_index in range(len(data)):
            if (data[cell_index].value is not None) and (len(data[cell_index].value) >= max_len):
                max_len = len(data[cell_index].value)
                max_len_index = cell_index
        return max_len_index

    def _get_cell_row_and_col(self, cell, line_length: int = 50):
        """
        根据单元格内容判断其具体需要多少行多少列
        """
        data = []

        # 匹配内容中的"\n"进行多行分割
        lines_data = cell.value.splitlines()

        # 根据line_length一行长度来进行按行分割
        for line in lines_data:
            data.extend([line[i:i+line_length]
                        for i in range(0, len(line), line_length)])
        return len(data), max(len(i) for i in data)

    def _set_col_width(self, sheet):
        """
        设置列宽
        """
        columns = list(sheet.columns)

        # 列遍历获取每列最长cell，修改单元格宽度
        for col_index in range(len(columns)):
            try:
                max_col_len_index = self._get_max_len_cell_index(columns[col_index])
                _, col_num = self._get_cell_row_and_col(columns[col_index][max_col_len_index])

                # 修改列宽
                if col_num < 50:
                    sheet.column_dimensions[get_column_letter(col_index + 1)].width = col_num * 2.5 + 2
                else:
                    sheet.column_dimensions[get_column_letter(col_index + 1)].width = 130
            except (ValueError, IndexError, AttributeError) as error:
                # 捕获在多个sheet的情况下，其余sheet内无数据导致的调用max()方法异常
                ...

    def _set_row_height(self, sheet, alignment: Alignment = alignment):
        """
        设置行高
        """
        rows = list(sheet.rows)

        # 行遍历获取没行最长cell，修改行高和超出长度单元格排布属性
        for row_index in range(len(rows)):
            try:
                max_row_len_index = self._get_max_len_cell_index(rows[row_index])
                row_num, _ = self._get_cell_row_and_col(rows[row_index][max_row_len_index])

                sheet.row_dimensions[row_index + 1].height = row_num * 20
                if row_num > 1:
                    for col_index in range(sheet.max_column):
                        # 修复文字内容过长情况下修改cell高度文字的排版情况
                        sheet[f"{get_column_letter(col_index + 1)}{row_index + 1}"].alignment = alignment
            except (ValueError, IndexError, AttributeError) as error:
                ...

    @classmethod
    def pref_excel_style(cls, file_path, border: Border = thin_border, alignment: Alignment = alignment):
        """
        优化Excel样式
            根据单元格内文字长度优化行高和列宽
            优化单元格边框样式
        """
        # 获取工作博
        workbook = load_workbook(file_path)

        # 根据sheet名称来获取当前工作表，逐sheet修改
        for sheetname in workbook.sheetnames:
            sheet = workbook[sheetname]

            cls()._set_row_height(sheet, alignment)
            cls()._set_col_width(sheet)

            # 所有数据添加单元格
            for row in sheet.rows:
                for cell in row:
                    cell.border = border

        workbook.save(file_path)

ExcelStyle.pref_excel_style(file_path)
```
以上的行高和列宽与单元格文字长度是通过设置Excel最适合的行高和最适合的列宽厚根据比例关系经过多次试验后得到了修改样式后显示效果较优的关系，可根据自身需求更改。