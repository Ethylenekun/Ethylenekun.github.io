---
title: Pandas习题-盖若(多级索引/数据合并)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas习题，在日常过程中遇见的比较不错的习题
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 3
abbrlink: 141de9b
date: 2025-11-18 16:04:51
---

# Pandas习题 - 盖若(多级索引/数据合并)	

## [指定列为基准相加两个数据](https://gairuo.com/m/pandas-adds-two-data-specified-column)

> 题目：以 x1 和 x2 为基础，将 value1 和 value2 两个列的对应数据相加，最终数据以先 df1 后 df2 为顺序的

```python
import io

data1 = '''
x1 x2 value1 value2
 A  C      1      2
 B  D      3      4
 C  E      5      6
'''

data2 = '''
 x1   x2  value1  value2
  A    E     1.0     2.0
  B    D     3.0     4.0
  C    F     5.0     6.0
  B  NaN     3.0     NaN
NaN    T     NaN     NaN
'''

df1 = pd.read_csv(io.StringIO(data1), sep=r'\s+')
df2 = pd.read_csv(io.StringIO(data2), sep=r'\s+')
```

> 解法

```python
df1.set_index(["x1","x2"],inplace=True)
df2.set_index(["x1","x2"],inplace=True)

(
    df1.add(df2,fill_value=0)
    .reindex(df1.index.append(df2.index))
    .reset_index()
)
```

---

## [对多层索引的部分列进行数字格式化](https://gairuo.com/m/pandas-multi-index-columns-formatting)

> 习题：对「毛利率」这一层以下的数据进行格式化，保留两位小数增加百分号，如 12.20%

```python
data = {
    ('SKU编码', 'len'): {('A', 'a1'): 19, ('A', 'a2'): 12},
    ('SKU编码', 'long'): {('A', 'a1'): 54, ('A', 'a2'): 75},
    ('毛利率', 'min'): {('A', 'a1'): 0.0588, ('A', 'a2'): 0.0909},
    ('毛利率', 'max'): {('A', 'a1'): 0.122, ('A', 'a2'): 0.1071},
    ('毛利率', 'mean'): {('A', 'a1'): 0.0841684210526316, ('A', 'a2'): 0.098475},
    ('毛利率', 'median'): {('A', 'a1'): 0.0827, ('A', 'a2'): 0.0991}
}

df = pd.DataFrame(data)
```

> 解法

```python
df.apply(lambda x:x.map("{:,.2%}".format) if x.name[0] == "毛利率" else x)
```

---

## [将 DataFrame 数据转为一行](https://gairuo.com/m/pandas-converts-dataframe-into-row)

> 题目：一个 DataFrame 结构的数据转为一行，即将二维的数据展开

```python
df = pd.DataFrame({
    'a': ['a1', 'a2', 'a3'],
    'b': [11, 22, 33],
    'c': [44, 55, 66]
})

# 所需结果
'''
    a   b   c   a   b   c   a   b   c
0  a1  11  44  a2  22  55  a3  33  66
'''
```

> 解法

```python
cols = df.stack().index.get_level_values(1)
values = df.stack().values

pd.DataFrame([values],columns=cols)

# 官方解法
df.stack().to_frame().droplevel(0).T
```

---

## [计算不同商品销售额日环比](https://gairuo.com/m/pandas-sales-ring-ratio-day)

> 题目：需要按类型先进行分组，在分组内按日期求得当前日期对比前一日期的变化比例

```python
from io import StringIO

data = """
时间	类型	销售额
2021/10/26	A	13.34827619
2021/10/26	B	86.92582545
2021/10/26	C	69.955569
2021/10/27	A	79.96022902
2021/10/27	B	83.94702123
2021/10/28	A	70.55949574
2021/10/28	B	40.0521712
2021/10/28	C	16.17825062
2021/10/29	A	12.8097718
2021/10/30	A	17.12582533
2021/10/30	C	91.12357503
2021/10/31	A	5.909534462
2021/10/31	B	78.56672937
2021/10/31	C	54.02725342
"""

df = pd.read_table(StringIO(data))
```

> 解法：`freq` 参数用于指定用于计算时序数据的频率

```python
(
    df.assign(环比 = 
        df.astype({"时间": "datetime64[ns]"})
        .set_index("时间")
        .groupby("类型")["销售额"]
        .transform(lambda x: x.pct_change(freq="D"))
        .reset_index(drop=True)
    )
)
```

---

## [将相同表头的 DataFrame 合并到一起](https://gairuo.com/m/pandas-merges-dataframes-same-header)

> 题目：在一批 DataFrame 中将有相同表头（列名）的 DataFrame 合并到一起

```python
df1 = pd.DataFrame({'a':[12],'d':[35]})
df2 = pd.DataFrame({'b':[13]})
df3 = pd.DataFrame({'a':[25],'d':[35]})
df4 = pd.DataFrame({'b':[28]})

dflist = [df1, df2, df3, df4]
```

> 解法

```python
(
    df.assign(col = df.data.map(tuple))
    .groupby('col')
    .data
    .agg(list)
    .apply(pd.concat)
)
```

---

## [根据指定列按列合并目录中的 Excel](https://gairuo.com/m/pandas-merges-excel-catalog)

> 题目：将实现按目录读取所有 Eexcel 文件，用 `pd.concat` 合并 Dataframe
>
> [数据集下载](https://gairuo.com/file/pic/2020/data_test.zip)
>
> - 读取目录所有 Excel 文件
> - 按 TM 进行合并，将其他文件的内容追加到原文件的后列
> - 新文件的列名（字段名）增加文件名的前缀，方便区分
> - 合并后数据按 TM 排序

```python
'''
                    TM       Z       Q
0     1995-01-04 08:00  160.02  4140.0
1     1995-01-05 08:00  159.93  4000.0
2     1995-01-06 08:00  160.05  4180.0
3     1995-01-07 08:00  159.93  4040.0
4     1995-01-08 08:00  159.88  3990.0
...                ...     ...     ...
2353  1998-04-03 08:00  158.52  2690.0
2354  1998-04-04 08:00  158.64  2800.0
2355  1998-04-05 08:00  158.72  2870.0
2356  1998-04-06 08:00  158.72  2910.0
2357  1998-04-07 08:00  158.58  2740.0
'''
```

> 解法

```python
import glob

files = glob.glob("data_test/*.xlsx")

f = lambda x:x.split(r"\\"[-1])[-1].split(".")[0] + "_"

dfs = [pd.read_excel(file).set_index("TM").rename(columns = lambda x:f(file)+x) for file in files]

pd.concat(dfs,axis=1).sort_index(level="TM")
'''
                   df1_Z   df1_Q  df2_Z   df2_Q
TM
1995-01-04 08:00  160.02  4140.0  39.59  5020.0
1995-01-05 08:00  159.93  4000.0  39.67  5160.0
1995-01-06 08:00  160.05  4180.0  39.93  5600.0
1995-01-07 08:00  159.93  4040.0  40.00  5740.0
1995-01-08 08:00  159.88  3990.0  39.98  5700.0
1995-01-09 08:00  159.83  3930.0  40.04  5800.0
1995-01-10 08:00  159.78  3880.0  39.83  5440.0
1995-01-10 09:30     NaN     NaN  39.93  5640.0
1995-01-10 14:18  159.82  3920.0    NaN     NaN
1995-01-11 08:00  159.55  3640.0  39.84  5450.0
'''
```

---

## [将连接字符串转为虚拟变量](https://gairuo.com/m/pandas-string-dummy-variables)

> 题目：需求期望将vote_code中的变量转为列，值为对应如果包含则为 1，否则为 0

```python
from io import StringIO

data = '''
投票区  vote_code
A选区   S5.S17.S6
B选区       S6.S9
C选区  S21.S5.S17
'''

df = pd.read_csv(StringIO(data), sep=r'\s+')

# 所需结果
'''
   投票区   vote_code  S17  S21  S5  S6  S9
0  A选区   S5.S17.S6    1    0   1   1   0
1  B选区       S6.S9    0    0   0   1   1
2  C选区  S21.S5.S17    1    1   1   0   0
'''
```

> 解法

```python
(
    df.vote_code.str.split(".")
    .explode()
    .pipe(pd.get_dummies)
    .groupby(level=0)
    .sum()
    .pipe(df.join)
)
```

---

