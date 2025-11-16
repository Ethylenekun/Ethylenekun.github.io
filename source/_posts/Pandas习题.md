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

# Pandas习题 - 盖若(入门/基础部分)

## [解析开始结束年份和最大间隔](https://gairuo.com/m/pandas-parsing-year-gap)

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

> 解法

```python
(
    df.Years.map(eval).explode()
    .groupby(level=0)
    .agg(from_year="min",to_year="max",largest_gap=lambda x:x.diff().max())
)
```

---

## [openpyxl完成Excel指定单元格样式](https://gairuo.com/m/pandas-openpyxl-excel-cell-styles)

> 题目：将 `'B5':'E10'` 范围的单元格设置为绿色背景、红色字体、14 号大小

> 解法

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

## [两个不同结构DataFrame相加合并](https://gairuo.com/m/pandas-addition-dataframes)

> 题目：将两个 DataFrame 合并，共有的索引标签 c2 行的值相加，非共有的 c1、c3 保持原值

```python
df1 = pd.DataFrame({'a':[1,2], 'b':[3,4]}, index=['c1','c2'])
df2 = pd.DataFrame({'a':[1,2], 'b':[3,4]}, index=['c2','c3'])
```

> 解法

1. ```python
   df1.add(df2,fill_value=0).astype(int)
   ```

2. ```python
   (df1 + df2).fillna(df1).fillna(df2).astype(int)
      
   # 类似方法
   (df1 + df2).combine_first(df1).combine_first(df2).astype(int)
   ```

---

## [根据文本数据提取特征](https://gairuo.com/m/pandas-text-extraction-features)

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
>  letter_dict = {}
>  
>  for letter in letters:
>      letter_dict[letter] = (letter_dict[letter] + 1 if letter in letter_dict else 1)
>      
>  return letter_dict
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

##  [按列拆分导出Excel](https://gairuo.com/m/pandas-split-column-export-excel)

> 题目：按列拆分Excel，批量将一个Excel拆分多个Excel

```python
import pandas as pd

df = pd.read_excel('https://gairuo.com/file/data/team.xlsx')
df = df.head()
df.columns = ['name'] + ['foo']*2 + ['bar']*3
df
'''
    name  foo  foo  bar  bar  bar
0  Liver    E   89   21   24   64
1   Arry    C   36   37   37   57
2    Ack    A   57   60   18   84
3  Eorge    C   93   96   71   78
4    Oah    D   65   49   61   86
'''
```

> 解法：因groupby中axis=1已弃用，故使用T方法进行转置

```python
df.set_index('name',inplace=True)

df.T.groupby(level=0).apply(lambda x:x.T.to_excel(f"{x.name}.xlsx"))
```

---

## [对每个工作簿增加工作簿名称的列](https://gairuo.com/m/pandas-column-workbook-name-excel)

> 题目：在工作簿的每个工作表数据内都插入一列，列的内容为对应工作的名称

```python
dfs = pd.read_excel("data/company.xlsx",sheet_name=None)
```

> 解法

```python
with pd.ExcelWriter("data/company.xlsx") as file:
    for sht_name,df in dfs.items():
        df.assign(company = sht_name).to_excel(file,sheet_name=sht_name,index=False)
```

---

## [标记按月连续变化趋势](https://gairuo.com/m/pandas-trend-continuous-change)

> 题目：要以6月为基础，看每个行 6 月前的数据相对 6 月是否有上升、下降或持平，还要说明持续时间

```python
import pandas as pd
import numpy as np

np.random.seed(4096)

df = pd.DataFrame(np.random.randint(0, 100, (8, 6)),
                  columns=pd.array([*'123456'])+'月'
                 )

df.iloc[4, 1:] = df.iloc[4,1]

# 结果需求

'''
   1月    2月    3月    4月    5月   6月     连续增减
0    36    40    89    40    90    88  连续1个月降低
1    23    78    70    62    30    17  连续4个月降低
2    23    42    68    49    87    96  连续2个月增长
3    66    75    45    96    61    20  连续2个月降低
4    22    71    71    71    71    71  连续4个月持平
5     2    41    64    72    89    96  连续5个月增长
6    35    93    45    42    75    18  连续1个月降低
7    99    83     1    50    85    64  连续1个月降低
'''
```

> 解法

```python
def f(ser:pd.Series):
    ser_diff = ser.diff().map(np.sign)
    status = {0:"持平",1:"增长",-1:"降低"}[ser_diff.iloc[-1]]
    cnt =  (ser_diff == ser_diff.iloc[-1])[::-1].cumprod().sum()
    return f"连续{cnt}个月{status}"

df.assign(连续增减 = df.apply(f,axis=1))
```

---

## [增加当前行与上行之和的新列](https://gairuo.com/m/pandas-new-column-sum-previous-row)

> 题目：增加一个 C 列。C 列的索引 0 取 B 列的对应值，其他值取 C 列自己上一个值与对应 A 列值的和

```python
import random

random.seed(666)
rng = range(100, 1100, 100)
data = {
    'A': [round(random.random(), 2) for i in rng],
    'B': rng
}

df = pd.DataFrame(data)

# 所需结果
'''
      A     B       C
0  0.46   100  100.00
1  0.90   200  100.90
2  0.43   300  101.33
3  0.50   400  101.83
4  0.81   500  102.64
5  0.55   600  103.19
6  0.71   700  103.90
7  0.12   800  104.02
8  0.81   900  104.83
9  0.39  1000  105.22
'''
```

> 解法

```python
df.assign(C = df.A.mask(df.index == 0,df.B).cumsum())
```

---

## [筛选日期前后连续的行](https://gairuo.com/m/pandas-finds-consecutive-dates-row)

> 题目：日期的连续要判断它们的间隔是一天

```python
from io import StringIO

data = '''
a
2022/9/23
2022/10/24
2022/9/21
2022/9/22
2022/9/23
2022/9/25
2022/9/26
2022/9/26
2022/11/2
2022/11/3
'''

df = pd.read_csv(StringIO(data))
```

> 解法

```python
df = df.astype("datetime64[ns]")

df[(df.a.diff() == pd.Timedelta("1d")) | (df.a.diff(-1).abs() == pd.Timedelta("1d"))]
```

---

## [筛选出指定样本数据](https://gairuo.com/m/pandas-selects-specified-sample)

> 题目：需要将数据集中的符合样本，数据的内容全部筛选出来

```python
import io

col = '''
col
3
4
7
3
6
3
4
7
3
5
7
3
4
7
'''

# 样本数据
data = [3,4,7]

# 读入数据
df = pd.read_csv(io.StringIO(col))
```

> 解法：按照窗口函数的值是否等于样本数据，窗口函数内的各个对象为DataFrame

```python
idx_list = []

for roller in df.rolling(window=3):
    if roller.col.to_list() == data:
        idx_list.extend(roller.index)
    
df.reindex(index = idx_list)
```

---

## [实现类似 SQL row_number分组行号](https://gairuo.com/m/pandas-sql-group-row-number)

> 题目：a 列为分组，b 列为值，需求为增加一个 row_number 列，显示按 a 分组的行号

```python
import random

random.seed(666)

df = pd.DataFrame({'a': [1,2,2,2,1,2,1,1],
                   'b': random.choices(range(10), k=8)
                  })
```

> 解法

```python
# 解法1
(
    df.groupby('a')['a']
    .transform('rank', method='first')
    .astype(int)
)

# 解法2
df.assign(rk = df.groupby("a").cumcount() + 1)
```

## [将列表中同符号的数据重新排序](https://gairuo.com/m/pandas-same-sign-list-order)

> 题目：在保证正负值段的整体顺序不变的情况下，在此段内容按绝对值从小到大排序

```python
l = [-8,-7, -2, -1, 1, 3, 2, -3 ,-1 ,-2 ,9 ,10, 2]

# 所需结果
[-1, -2, -7, -8, 1, 2, 3, -1, -2, -3, 2, 9, 10]
```

> 解法

```python
df = pd.DataFrame({"num":l})

ser = df["num"].map(np.sign)

(
    df.assign(gb=ser.ne(ser.shift()).cumsum())
    .groupby("gb")["num"]
    .apply(lambda x:x.sort_values(ascending=False) if x.sum() < 0 else x.sort_values()) # 判断组内是负数组还是正数组来决定排序方向
    .to_list()
)
```

---

## [按顺序填充缺失值](https://gairuo.com/m/pandas-fill-missing-order)

> 题目：按照从上到下的顺序，将 y 列的 3 个有效值填充到 x 列的缺失位上

```python
df = pd.DataFrame({'x': list('aabbccdd'),
                   'y': list('ABC')+5*[np.nan]})
# 创造缺失值
df.iloc[np.r_[2, 3, 5], 0] = np.nan
df
'''
     x    y
0    a    A
1    a    B
2  NaN    C
3  NaN  NaN
4    c  NaN
5  NaN  NaN
6    d  NaN
7    d  NaN
'''
```

> 解法

```python
# 解法1:循环填充x列
for i,j in enumerate(df["x"][df["x"].isna()].index):
    df.loc[j,"x"] = df["y"][df["y"].notna()][i]
    
# 解法2:构建y列非空值的x列空值索引的新Series
ser = df["y"][df["y"].notna()].set_axis(df["x"][df["x"].isna()].index)
df["x"].fillna(ser)
```

---

## [统计设备运行天数](https://gairuo.com/m/pandas-device-uptime-days-online)

> 题目：1 为运行状态，0 为停机状态。我们需要统计每次运行状态的最后一天及运行的天数

```python
data = {
    "Date": [
        "5/29/2025", "5/31/2025", "6/1/2025", "6/2/2025",
        "6/3/2025", "6/4/2025", "6/5/2025", "6/6/2025",
        "6/7/2025", "6/8/2025", "6/9/2025", "6/10/2025"
    ],
    "Status": [1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1]
}
df = pd.DataFrame(data)
df["Date"] = pd.to_datetime(df["Date"])
df
'''
         Date  Status
0  2025-05-29       1
1  2025-05-31       1
2  2025-06-01       1
3  2025-06-02       1
4  2025-06-03       0
5  2025-06-04       0
6  2025-06-05       0
7  2025-06-06       1
8  2025-06-07       1
9  2025-06-08       1
10 2025-06-09       1
11 2025-06-10       1
'''
```

> 解法

```python
(
    df.assign(gb = df["Status"].ne(df["Status"].shift()).cumsum())
    .query("Status > 0")
    .groupby("gb",as_index=False)["Date"]
    .agg(Date = "max",DaysOnline = "count")
    .drop("gb",axis=1)
)
```

---

## [按活动时间范围合并数据](https://gairuo.com/m/pandas-merge-activity-range)

> 题目：在销售表的基础上，将活动表的三个信息增加成为其后的列，当然这个要根据活动的时间范围

```python
df_sales = pd.DataFrame({
        '销售日期':pd.date_range('20240101', periods=9),
        '销售额':np.random.randint(100, 1000, 9)
})

df_campiagn = pd.DataFrame({
    '活动开始日期':[pd.to_datetime('20240102'), pd.to_datetime('20240105')],
    '活动结束日期':[pd.to_datetime('20240103'), pd.to_datetime('20240107')],
    '促销描述':['买一送一', '打8折']
})

# 所需结果

'''
  销售日期  销售额  活动开始日期 活动结束日期  促销描述
0 2024-01-01  535        NaT        NaT   NaN
1 2024-01-02  237 2024-01-02 2024-01-03  买一送一
2 2024-01-03  251 2024-01-02 2024-01-03  买一送一
3 2024-01-04  983        NaT        NaT   NaN
4 2024-01-05  989 2024-01-05 2024-01-07   打8折
5 2024-01-06  999 2024-01-05 2024-01-07   打8折
6 2024-01-07  549 2024-01-05 2024-01-07   打8折
7 2024-01-08  648        NaT        NaT   NaN
8 2024-01-09  236        NaT        NaT   NaN
'''
```

> 解法

```python
for i,j in df_sales.iterrows():
    dt = j["销售日期"]
    df = df_campiagn.query("(@dt >= 活动开始日期) & (@dt <= 活动结束日期)")
    
    if len(df) > 0:
        df_sales.loc[i, "活动开始日期"] = df.iloc[0]["活动开始日期"]
        df_sales.loc[i, "活动结束日期"] = df.iloc[0]["活动结束日期"]
        df_sales.loc[i, "促销描述"] = df.iloc[0]["促销描述"]
```

---

## [非向量化修改数据](https://gairuo.com/m/pandas-non-vectorized-calculation)

> 题目：需要增加一列 c，c 列的计算逻辑是第一行的值为所在行的a+b，第二行及以后的值为当前行的a + 上一行的c

```python
df = pd.DataFrame({'a': [5, 6, 7], 'b': [3, 5, 8]})
df

# 所需结果
'''
   a  b   c
0  5  3   8
1  6  5  14
2  7  8  21
'''
```

> 习题

```python
for i,j in df.iterrows():
    if i == 0:
        df.loc[i,"c"] = df.loc[i,["a","b"]].sum()
    else:
        df.loc[i,"c"] = df.loc[i-1,"c"] + j.a
```

---

## [计算实验仪器正常运行周期时长](https://gairuo.com/m/pandas-normal-period-experimental-instruments)

> 题目：增加 Delta 列来计算显示 T（触发）和 C（结束）状态之间的时间差，状态 C 之后不经过 T 还可能出现状态 C，我们视 T 之后到第一个 C 之间为一个正常运行周期

```python
df=pd.DataFrame(columns=['Status','t'])

df['Status']=['A','T', 'A','C','C','A','T','T','C','A']
df['t']= pd.to_datetime(['2021-06-12 08:39:24.813000',
'2021-06-12 08:39:24.820000',
'2021-06-12 08:39:25.210000',
'2021-06-12 08:39:25.217000',
'2021-06-12 08:44:28.830000',
'2021-06-12 10:48:10.293000',
'2021-06-12 10:48:10.300000',
'2021-06-12 10:48:10.680000',
'2021-06-12 10:48:10.693000',
'2021-06-12 10:48:11.223000'])

# 所需结果

'''
  Status                       t                  Delta
0      A 2021-06-12 08:39:24.813                    NaT
1      T 2021-06-12 08:39:24.820                    NaT
2      A 2021-06-12 08:39:25.210                    NaT
3      C 2021-06-12 08:39:25.217 0 days 00:00:00.397000
4      C 2021-06-12 08:44:28.830                    NaT
5      A 2021-06-12 10:48:10.293                    NaT
6      T 2021-06-12 10:48:10.300                    NaT
7      T 2021-06-12 10:48:10.680                    NaT
8      C 2021-06-12 10:48:10.693 0 days 00:00:00.013000
9      A 2021-06-12 10:48:11.223                    NaT
'''
```

> 解法

```python
temp = (
    df.query("Status in ['T','C']")
    .assign(next_t=(lambda x: (x["Status"] == "C") & (x["Status"].shift() == "T")))
    .assign(prev_c=(lambda x: (x["Status"] == "T") & (x["Status"].shift(-1) == "C")))
    .query("(next_t == True) | (prev_c == True)")
    .assign(Delta = lambda x:x.t.diff())
    .query("Status == 'C'")
)

for i,j in temp.iterrows():
    df.loc[i,"Delta"] = j["Delta"]
```

---

## [判断当前值是否在本行之前出现过](https://gairuo.com/m/pandas-current-appeared-before)

> 题目：需要增加一列，如果这个数据之前没出现过记为 True，如果出现过记为 False

```python
df = pd.DataFrame({"ID": [123, 234, 123, 234, 678]})
```

> 解法

```python
df.assign(是否是新ID = df.index.map(lambda x:df.loc[x,"ID"] not in df.loc[:x-1,"ID"].values))

# 性能较好
df.assign(是否是新ID=~df.duplicated(["ID"], keep="first"))
```

---

