---
title: Pandas习题(多层索引、数据合并、数据清洗)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas习题，在日常过程中遇见的比较不错的习题
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 3
abbrlink: 9d2692f0
---

# Pandas习题 - 多级索引/数据合并/数据清洗

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

# 使用pd.concat
pd.concat([df.loc[[idx],:].set_axis([0]) for idx in df.index],axis=1)

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

## [将列表转为以字符为键的字典](https://gairuo.com/m/pandas-list-character-dictionary)

> 题目：键是前边的字母，值是字母后边的数字，如果有多个字母则是一个列表

```python
m = ['a',1,'b',13,14,'d',67]

# 所需结果
{'a': 1, 'b': [13, 14], 'd': 67}
```

> 解法

```python
df = pd.DataFrame({"x": m})

(
    df.assign(y=df.x.where(df.x.str.isalpha()))
    .assign(z=lambda x: x.y.ffill())
    .query("x!=z")
    .groupby("z")
    .x.agg(lambda x: list(x) if len(x) > 1 else x)
    .to_dict()
)
```

---

## [缺失值填充为分组的平均值](https://gairuo.com/m/pandas-missing-values-filled-mean-group)

> 题目：将这两个缺失值按其所在组的平均值进行填充

```python
df = pd.read_csv('https://gairuo.com/file/data/team.csv')

# 创建缺失值
# 创建缺失值
df = df.head(10).loc[:,'name':'Q1']
df.loc[(1, 2), 'Q1'] = None
df
```

> 解法

```python
# 个人解法
gb = df.groupby("team").agg({"Q1":"mean"})
df.set_index('team').fillna(gb).reset_index()

# 官方解法1
df.apply(lambda x:x.fillna(df.groupby('team').agg({"Q1":"mean"}).loc[x["team"]]),axis=1)

# 官方解法2
mapping = df.groupby('team').Q1.transform("mean")
df.assign(Q1 = df.Q1.mask(df.Q1.isna(),mapping))
```

---

## [根据树状文本解析其上级](https://gairuo.com/m/pandas-parses-parent-tree-text)

> 题目

```python
from io import StringIO

data = """
class_
A
——B1
——B2
————C1
——B3
————C2
————C3
"""

df = pd.read_csv(StringIO(data))

# 所需结果
'''
   0     1     2         foo   res
0  A  None  None          [A]  None
1  A    B1  None      [A, B1]     A
2  A    B2  None      [A, B2]     A
3  A    B2    C1  [A, B2, C1]    B2
4  A    B3  None      [A, B3]     A
5  A    B3    C2  [A, B3, C2]    B3
6  A    B3    C3  [A, B3, C3]    B3
'''
```

> 解法

```python
(
    df.class_.str.split('——', expand=True)
    .replace({'': None})
    .apply(lambda x: x if x.name == 2 else x.ffill())
    .assign(foo=lambda x: x.apply(lambda s: s.dropna().to_list(), axis=1))
    .assign(res=lambda x: x.foo.apply(lambda s: s[-2] if len(s)>1 else None))
)
```

---

## [筛选日期连续的数据行](https://gairuo.com/m/pandas-filters-consecutive-dates)

> 题目：筛选出不分位置日期连续的行

```python
from io import StringIO

data = '''
a,b,c
2022/9/23,2022/9/24,2022/9/27
2022/9/26,2022/9/27,2022/9/23
2022/9/21,2022/9/22,2022/9/26
2022/9/26,2022/9/22,2022/9/21
2022/9/26,2022/9/24,2022/9/27
2022/9/22,2022/9/21,2022/9/25
2022/9/26,2022/9/21,2022/9/27
2022/9/26,2022/9/21,2022/9/23
2022/9/25,2022/9/24,2022/9/27
2022/9/23,2022/9/25,2022/9/26
2022/9/27,2022/9/26,2022/9/23
2022/9/25,2022/9/24,2022/9/26
2022/9/25,2022/9/24,2022/9/27
2022/9/21,2022/9/26,2022/9/22
2022/9/24,2022/9/25,2022/9/26
2022/9/25,2022/9/21,2022/9/26
2022/9/24,2022/9/26,2022/9/23
2022/9/26,2022/9/22,2022/9/21
2022/9/22,2022/9/27,2022/9/23
2022/9/27,2022/9/21,2022/9/25
2022/9/27,2022/9/25,2022/9/26
2022/9/26,2022/9/24,2022/9/23
2022/9/25,2022/9/26,2022/9/23
2022/9/25,2022/9/21,2022/9/26
2022/9/23,2022/9/25,2022/9/26
2022/9/22,2022/9/26,2022/9/23
2022/9/26,2022/9/27,2022/9/21
2022/9/25,2022/9/21,2022/9/24
2022/9/26,2022/9/21,2022/9/27
2022/9/22,2022/9/21,2022/9/27
2022/9/27,2022/9/26,2022/9/23
2022/9/22,2022/9/26,2022/9/27
2022/9/22,2022/9/27,2022/9/26
'''

df = pd.read_csv(StringIO(data))

# 所需结果
'''
            a          b          c
11  2022/9/25  2022/9/24  2022/9/26
14  2022/9/24  2022/9/25  2022/9/26
20  2022/9/27  2022/9/25  2022/9/26
'''
```

> 解法

```python
def func(ser: pd.Series) -> bool:
    return (
        ser.astype("datetime64[ns]").sort_values().diff().dropna().eq(pd.Timedelta("1days")).all()
    )

df[df.apply(func, axis=1)]
```

---

## [根据值在其他列中出现的次数指定列值](https://gairuo.com/m/pandas-column-depending-times-other)

> 题目：增加一个名为“Temperature”的新列，当 Container 第一次出现Event为“Clean”时，它的值为4，而所有后续的“Clean“事件的值为3。Dry时的值总是1

```python
import io

data = '''
Container Event
A         Clean
B         Dry
A         Clean
A         Dry
B         Clean
C         Clean
C         Clean
C         Clean
'''

df = pd.read_csv(io.StringIO(data), sep=r'\s+')

# 所需结果
'''
Container  Event  Temperature
A          Clean  4
B          Dry    1
A          Clean  3
A          Dry    1
B          Clean  4
C          Clean  4
C          Clean  3
C          Clean  3
'''
```

> 解法

```python
# 官方解法: 使用np.select
cond1 = ~df.duplicated()
cond2 = df['Event'].eq('Clean')
cond3 = df['Event'].eq('Dry')
df['Temperature'] = np.select([cond1 & cond2, cond2, cond3], [4, 3, 1])

# 个人解法
df.assign(
    Temperature=df.groupby("Container")["Event"]
    .transform(lambda x:x.mask(x=="Dry",1).mask(x != "Dry",4-x.duplicated()))
)
```

---

## [将时间区间展开为年月两列](https://gairuo.com/m/pandas-expands-interval-month-year)

> 题目：将数据按起始日期到结束日期展开，增加年和月列

```python
from io import StringIO

data = '''
合同编号    起始日期    结束日期
BH001   2023-02-20  2024-02-19
BH002   2023-04-01  2026-03-31
BH003   2022-04-10  2023-04-09
BH004   2023-03-01  2024-09-30
BH005   2023-02-01  2026-01-31
'''

df = pd.read_csv(StringIO(data), sep=r'\s+')

# 所需结果
'''
合同编号    年   月
BH001   2023    2
BH001   2023    3
BH001   2023    4
BH001   2023    5
BH001   2023    6
BH001   2023    7
BH001   2023    8
BH001   2023    9
BH001   2023    10
BH001   2023    11
BH001   2023    12
BH001   2024    1
BH001   2024    2
'''
```

> 解法

```python
(
    df.assign(date=df.apply(lambda x: pd.date_range(x["起始日期"], x["结束日期"]), axis=1))
    .explode("date")
    .assign(年=lambda x: x["date"].dt.year, 月=lambda x: x["date"].dt.month)
    .loc[:, ["合同编号", "年", "月"]]
    .drop_duplicates()
)
```

---

