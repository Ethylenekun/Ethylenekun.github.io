---
title: Pandas习题
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas习题，在日常过程中遇见的比较不错的习题
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 2
abbrlink: c02b5fa5
date: 2025-11-09 16:29:02
---

# Pandas习题

## [pandas 解析开始结束年份和最大间隔](https://gairuo.com/m/pandas-parsing-year-gap)

> 题目：在一个列表形式的数据中，来解析开始和结束年份以及它们的最大年份间隔

```python
import io

data = '''
Years
[1990,1991,1995,2000,2001,2006]
[1990,1990]
[1980,1981,1990,1995]
'''

df = pd.read_csv(io.StringIO(data), sep=r'\s+')


# 需求结果

'''
   from_year  to_year  largest_gap
0       1990     2006            5
1       1990     1990            0
2       1980     1995            9
'''
```

> 解法：

```python
(
    df.Years.map(eval).explode()
    .groupby(level=0)
    .agg(from_year="min",to_year="max",largest_gap=lambda x:x.diff().max())
)
```

---

## [pandas 配合 openpyxl 完成 Excel 指定单元格样式](https://gairuo.com/m/pandas-openpyxl-excel-cell-styles)

> 题目：将 `'B5':'E10'` 范围的单元格设置为绿色背景、红色字体、14 号大小

> 解法：

```python
from openpyxl import load_workbook
from openpyxl.styles import Font, PatternFill, colors

# 加载 Excel 文件
workbook = load_workbook('team.xlsx')

# 选择工作表
sheet = workbook.active

# 选取单元格范围
df = pd.DataFrame(sheet['B5':'E10'])

def modify_style(cell):
    # 设置样式
    bold_font = Font(size=14, bold=True, color='FF0000')
    yellow_fill = PatternFill(start_color='00FF00',
                          end_color='00FF00',
                          fill_type='solid')
    # 修改样式
    cell.font = bold_font
    cell.fill = yellow_fill

# 应用修改样式
df.map(modify_style)

# 保存修改后的 Excel 文件
workbook.save('team_formatted.xlsx')

# 关闭 Excel 文件
workbook.close()
```

---

## [pandas 两个不同结构 DataFrame 相加合并](https://gairuo.com/m/pandas-addition-dataframes)

> 题目：将两个 DataFrame 合并，共有的索引标签 c2 行的值相加，非共有的 c1、c3 保持原值

```python
df1 = pd.DataFrame({'a':[1,2], 'b':[3,4]}, index=['c1','c2'])
df2 = pd.DataFrame({'a':[1,2], 'b':[3,4]}, index=['c2','c3'])
```

> 解法：

1. ```python
   df1.add(df2,fill_value=0).astype(int)
   ```

2. ```python
   (df1 + df2).fillna(df1).fillna(df2).astype(int)
   
   # 类似方法
   (df1 + df2).combine_first(df1).combine_first(df2).astype(int)
   ```

---

## [pandas 根据文本数据提取特征](https://gairuo.com/m/pandas-text-extraction-features)

> 题目：在数据后边增加一些列，每个字母为一列，值为本行对应字母的数量

```python
df = pd.DataFrame({'a': ['6R-5A-4G-7R',
                         '2A-4G-3A',
                         '8G',
                         '1R-9A']
                  })
df
'''
             a
0  6R-5A-4G-7R
1     2A-4G-3A
2           8G
3        1R-9A
'''

# 所需结果
'''
             a   R  A  G
0  6R-5A-4G-7R  13  5  4
1     2A-4G-3A   0  5  4
2           8G   0  0  8
3        1R-9A   1  9  0
'''
```

> 解法：利用`Counter`返回字符串的字符和出现次数的字典

> 手动实现`Counter`效果
>
> ```python
> def My_Counter(letters:str) -> dict:
>     letter_dict = {}
>     
>     for letter in letters:
>         letter_dict[letter] = (letter_dict[letter] + 1 if letter in letter_dict else 1)
>         
>     return letter_dict
> ```

```python
from collections import Counter

ser = (
    df.a.map(lambda x:x.split("-"))
    .map(lambda i:"".join([j[1] * int(j[0]) for j in i]))
    .map(Counter)
)

df2 = pd.DataFrame(ser.to_list()).fillna(0).astype(int)

df.merge(df2,left_index=True,right_index=True)
```

---

