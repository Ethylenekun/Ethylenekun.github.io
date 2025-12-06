---
title: Pandas习题(分组聚合、重塑透视)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas习题，在日常过程中遇见的比较不错的习题
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 4
abbrlink: 1bf93122
---

# Pandas习题 - 分组聚合/重塑透视

## [连续打卡（对连续相同的数字计数）](https://gairuo.com/m/pandas-counting-consecutive-value)

> 题目：对部分连续的数据标记序数。比如，011101100，将连续的111，标记为123

```python
s = pd.Series([1,0,1,1,1,0,0,1,1,1])

# 所需结果
'''
0    1
1    0
2    1
3    2
4    3
5    0
6    0
7    1
8    2
9    3
dtype: int64
'''
```

> 解法

```python
s.groupby(s.ne(s.shift()).cumsum()).cumsum()
```

---

## [保留分组求和后的每组最大值](https://gairuo.com/m/pandas-keeps-maximum-group-summing)

> 题目：每天可能会生产不同产品多个批次，需求是得到每天最大数量的那批产品

```python
from io import StringIO

data = """
日期	产品	数量
2020/6/1	A	8
2020/6/1	A	7
2020/6/1	A	13
2020/6/1	B	13
2020/6/1	B	13
2020/6/1	D	15
2020/6/2	A	8
2020/6/2	A	2
2020/6/2	B	8
2020/6/2	C	13
2020/6/3	B	9
2020/6/3	B	3
2020/6/3	B	13
2020/6/3	B	6
2020/6/3	C	4
2020/6/3	D	13
2020/6/3	D	12
2020/6/3	D	12
2020/6/3	D	6
"""

df = pd.read_csv(StringIO(data),sep="\t")

# 所需结果
'''
             数量
日期       产品
2020/6/1 A   28
2020/6/2 C   13
2020/6/3 D   43
'''
```

> 解法

```python
# 官方解法
(
    df.groupby(['日期', '产品'])
    .sum()
    .loc[lambda x: x.groupby(level=0).idxmax().数量]
)

# 个人解法
(
    df.assign(数量 = df.groupby(["日期","产品"]).transform("sum"))
    .groupby('日期')
    .apply(lambda x:x.nlargest(1,"数量"))
    .set_index(["日期","产品"])
)
```

---

## [组内排序并保持小计位置](https://gairuo.com/m/pandas-sort-within-group-subtotal)

> 题目：需要按照「销售额」在组内进行排序，同时不能改变产品列小计的位置，让它一直在组的最底部

```python
df = pd.DataFrame({
    '楼层号': ['F01']* 4 + ['F02']*4,
    '产品' : (list('ABC') + ['小计'])*2,
    '销售额' : [2,3,1,6,7,9,8,24]
})

# 所需结果
'''
   产品  销售额
1   B    3
0   A    2
2   C    1
3  小计    6
5   B    9
6   C    8
4   A    7
7  小计   24
'''
```

> 解法

```python
(
    df.groupby("楼层号", as_index=False, group_keys=False)
    .apply(lambda x: pd.concat([x.iloc[:-1].sort_values("销售额", ascending=False), x.iloc[[-1]]]))
)
```

---

## [按组跳过指定值标记序号](https://gairuo.com/m/pandas-skip-specified-tag-group)

> 题目：增加一个编号列以标记对应行在商品分组内的顺序，其中要跳过活动名称为「正价」的行，并「正价」所在的行记为「#」
>
> **[扩展：case_when() 条件替换](https://gairuo.com/p/pandas-case-when)**

```python
df = pd.DataFrame({
    '商品' : ['A']*4 + ['B'] + ['C']*2 + ['D']*2,
    '活动名称' : ['买赠','正价','满减','六折',
              '正价','半价','六折','正价','六折']
 })

# 所需结果
'''
  商品 活动名称 编号
0  A   买赠  1
1  A   正价  #
2  A   满减  2
3  A   六折  3
4  B   正价  #
5  C   半价  1
6  C   六折  2
7  D   正价  #
8  D   六折  1
'''
```

> 解法

```python
ser = (
    df.groupby("商品",as_index=False,group_keys=False)["活动名称"]
    .apply(lambda x:(x != "正价").cumsum())
    .case_when([(df["活动名称"] == "正价","#")])
)

df.assign(编号 = ser)
```

---

## [按组生成分数表示数据](https://gairuo.com/m/pandas-generates-fractional-group)

> 题目：A 列为分组，B 列为指标，其中最后一个指标 D 为总体，C 列为指标的值

```python
import random

random.seed(7890)
data = {
    'A': ['x']*4 + ['y']*4,
    'B': [*'abcD']*2,
    'C': [random.randint(1,9) for i in range(8)],
}

df = pd.DataFrame(data)

# 所需结果
'''
   A    B    C
0  x  a_D  4/7
1  x  b_D  7/7
2  x  c_D  1/7
4  y  a_D  3/6
5  y  b_D  5/6
6  y  c_D  2/6
'''
```

> 解法

```python
# 官方解法
def func(grp: pd.DataFrame):

    b, c = grp.B, grp.C.astype(str)

    b = b.iloc[:-1] + '_' + b.iat[-1]
    c = c.iloc[:-1] + '/' + c.iat[-1]

    return pd.DataFrame({'A': grp.name, 'B': b, 'C': c})

df.groupby('A', group_keys=False).apply(func)

# 个人解法
(
    df.groupby("A", as_index=False, group_keys=False).apply(
        lambda x: x.iloc[:-1].assign(
            B=lambda i: i["B"] + "_" + x["B"].iloc[-1],
            C=lambda i: i["C"].map(str) + "/" + str(x["C"].iloc[-1]),
        )
    )
)
```

---

## [删除连续6个以上小于0.4的值](https://gairuo.com/m/pandas-deletes-than-six-consecutive)

> 题目：

```python
'''
2019/1/1 0:00 2.5 
2019/1/1 0:10 2.4 
2019/1/1 0:20 2.6 
2019/1/1 0:30 2.7 
2019/1/1 0:40 2.9 
2019/1/1 0:50 0.3 
2019/1/1 1:00 0.35 
2019/1/1 1:10 1.8 
2019/1/1 1:20 2.9 
2019/1/1 1:30 2.7 
2019/1/1 1:40 0.2 
2019/1/1 1:50 0.1 
2019/1/1 2:00 0.25 
2019/1/1 2:10 0.31 
2019/1/1 2:20 0.24 
2019/1/1 2:30 0.24 
2019/1/1 2:40 0.24 
2019/1/1 2:50 0.21 
2019/1/1 3:00 3.1 
2019/1/1 3:10 3.2 
2019/1/1 3:20 2.9
'''

# 复制上列数据
df = (pd.read_clipboard(header=None,
                       parse_dates={'time':[0,1]})
      .rename(columns={2: 'x'})
)
```

> 解法

```python
ser = df.x.gt(0.4)

(
    df.assign(gb = ser.ne(ser.shift()).cumsum())
    .groupby("gb")
    .filter(lambda y:~((y["x"].lt(0.4).all()) & (y['x'].count() >= 6)))
)
```

---

## [将分组名称插入本组内容之前](https://gairuo.com/m/pandas-inserts-group-name)

> 题目：将 a 插入 a 组的第一个 b 前边，将 e 插入 e 组的第一个 f 前边

```python
a b
a c
a d
e f
e g
e h

# 转换为：
a
b
c
d
e
f
g
h
```

> 解法

```python
# 个人解法
df.groupby("x").apply(
    lambda x: pd.concat([pd.Series(x.name), x["y"]])
).reset_index(drop=True)

# 官方解法
(df.groupby('x')
 .agg({'x': set, 'y': list})
 .map(list)
 .pipe(lambda d: d.x + d.y)
 .explode('y')
)
```

---

## [筛选出同组的相反数](https://gairuo.com/m/pandas-opposite-same-group)

> 题目：需要把在同一个分组中有相反数（数字相同符号不同）的数据筛选出来

```python
from io import StringIO

data = """
分组  数字
A1  12
A1  -12
A1  7
B2  8
B2  10
B2  -8
B2  -6
C3  5
C3  3
C3  14
D4  -5
D4  11
D4  11
"""

df = pd.read_csv(StringIO(data),sep=r"\s+")
# 所需结果
'''
   分组  数字           grp  negative
0  A1  12     [12, -12, 7]      True
1  A1 -12     [12, -12, 7]      True
3  B2   8  [8, 10, -8, -6]      True
5  B2  -8  [8, 10, -8, -6]      True
'''
```

> 解法

```python
(
    df.assign(grp=df.分组.apply(lambda x: df.groupby('分组').get_group(x).数字.to_list()))
    .assign(negative=lambda y: y.apply(lambda x: np.negative(x.数字) in x.grp, axis=1))
    .query('negative == True')
)
```

---

## [连续登录留存数据分析](https://gairuo.com/m/pandas-continuous-login)

> 题目：用户的所有登录日期是连续的则认为用户到访一次，记为 1，如果日期不连续则认为到访为多次，记为 2

```python
df = pd.DataFrame({
    'name': list('AAAABBACC'),
    'date': ['2021-1-1', '2021-1-2', '2021-1-3',
             '2021-1-4', '2021-3-5', '2021-3-21',
             '2021-2-22', '2021-3-8', '2021-3-9', ]
})

# 所需结果
'''
name
A    2
B    2
C    1
dtype: int64
'''
```

> 解法

```python
(
    df.astype({'date': 'datetime64[ns]'}) # 将数据转为时间类型
    .sort_values(['name', 'date']) # 排序，用户第一序，时间第二序
    .groupby('name') # 按用户分组
    # 对组内（每个用户）的时间做差，如果全为1说明连续
    .apply(lambda x: (x.date.diff(1).dt.days.fillna(1)==1).all())
    .map({True: 1, False: 2}) # 枚举映射为 1 和 2
)
```

---

## [扩展：pd.Grouper() 分组器](https://gairuo.com/p/pandas-grouper)

> 参数：
>
> - **key** (`str`, 默认为 `None`)：
>   指定用于分组的列名称。如果指定了 `key`，`Grouper` 将按照该列进行分组。
> - **level** (`int` 或 `str`, 默认为 `None`)：
>   指定用于分组的索引级别的名称或位置。在多级索引的情况下，选择某个级别进行分组。
> - **freq** (`str` 或 `DateOffset`, 默认为 `None`)：
>   指定时间序列分组的频率，如 `'D'`（天）、`'W'`（周）、`'M'`（月）等。此参数仅在 `key` 或 `level` 是日期时间对象时有效。
> - **axis** (`int` 或 `str`, 默认为 `0`)：
>   指定进行分组的轴。`0` 表示按行分组，`1` 表示按列分组。
> - **sort** (`bool`, 默认为 `False`)：
>   是否对结果标签进行排序。如果为 `True`，则对分组结果进行排序；否则不排序。
> - **closed** (`'left'` 或 `'right'`, 默认为 `None`)：
>   在时间序列分组时，指定区间是左闭还是右闭。如果未指定 `freq` 参数，此参数无效。
> - **label** (`'left'` 或 `'right'`, 默认为 `None`)：
>   指定区间的标记方式，`'right'` 表示右边界，`'left'` 表示左边界。仅在 `freq` 参数指定时有效。
> - **convention** (`'start'`, `'end'`, `'e'`, `'s'`, 默认为 `'start'`)：
>   如果 `grouper` 是 `PeriodIndex` 并且指定了 `freq` 参数，指定分组周期的开始或结束。
> - **origin** (`Timestamp` 或 `str`, 默认为 `'start_day'`)：
>   调整分组的起点时间戳。可选值包括：
>   - `'epoch'`：从 1970-01-01 开始
>   - `'start'`：时间序列的第一个值
>   - `'start_day'`：时间序列的第一个午夜
>   - `'end'`：时间序列的最后一个值
>   - `'end_day'`：时间序列的最后一天的午夜
> - **offset** (`Timedelta` 或 `str`, 默认为 `None`)：
>   添加到原点的时间偏移量。
> - **dropna** (`bool`, 默认为 `True`)：
>   是否丢弃包含 NA 值的组。如果为 `True`，包含 NA 的组将被丢弃；如果为 `False`，NA 值将作为组键处理。

```python
df.groupby(pd.Grouper(key="Animal"))
# 等同于
df.groupby('Animal')
```

```python
# 日期分组
df = pd.DataFrame(
   {
       "Publish date": [
            pd.Timestamp("2000-01-02"),
            pd.Timestamp("2000-01-02"),
            pd.Timestamp("2000-01-09"),
            pd.Timestamp("2000-01-16")
        ],
        "ID": [0, 1, 2, 3],
        "Price": [10, 20, 30, 40]
    }
)
df.groupby(pd.Grouper(key="Publish date", freq="1W")).mean()
'''
               ID  Price
Publish date
2000-01-02    0.5   15.0
2000-01-09    2.0   30.0
2000-01-16    3.0   40.0
'''
```

```python
start, end = '2000-10-01 23:30:00', '2000-10-02 00:30:00'
rng = pd.date_range(start, end, freq='7min')
ts = pd.Series(np.arange(len(rng)) * 3, index=rng)
ts
'''
2000-10-01 23:30:00     0
2000-10-01 23:37:00     3
2000-10-01 23:44:00     6
2000-10-01 23:51:00     9
2000-10-01 23:58:00    12
2000-10-02 00:05:00    15
2000-10-02 00:12:00    18
2000-10-02 00:19:00    21
2000-10-02 00:26:00    24
Freq: 7min, dtype: int64
'''

ts.groupby(pd.Grouper(freq='17min')).sum()
'''
2000-10-01 23:14:00     0
2000-10-01 23:31:00     9
2000-10-01 23:48:00    21
2000-10-02 00:05:00    54
2000-10-02 00:22:00    24
Freq: 17min, dtype: int64
'''

ts.groupby(pd.Grouper(freq='17min', origin='epoch')).sum()
'''
2000-10-01 23:18:00     0
2000-10-01 23:35:00    18
2000-10-01 23:52:00    27
2000-10-02 00:09:00    39
2000-10-02 00:26:00    24
Freq: 17min, dtype: int64
'''

ts.groupby(pd.Grouper(freq='17min', origin='2000-01-01')).sum()
'''
2000-10-01 23:24:00     3
2000-10-01 23:41:00    15
2000-10-01 23:58:00    45
2000-10-02 00:15:00    45
Freq: 17min, dtype: int64
'''
```

---

## [列表按间隔分组](https://gairuo.com/m/pandas-list-grouped-interval)

> 题目：从小到大排序后，将数字连续间隔小于为1或相等的数放在一起，形成一个新的列表

```python
numbers = [3, 6, 22, 0, 25, 1, 7, 5, 2, 13, 14, 26, 27]
df = pd.DataFrame({"n": numbers})
# 所需结果
# [[0, 1, 2, 3], [5, 6, 7], [13, 14], [22], [25, 26, 27]]
```

> 解法

```python
# 官方解法1
(
    pd.DataFrame({"n": [3, 6, 22, 0, 25, 1, 7, 5, 2, 13, 14, 26, 27]})
    .sort_values("n", ignore_index=True)
    .assign(sign=lambda x: x.index)
    .assign(sign=lambda x: np.where(x.n.diff(1) > 1, x.sign, np.nan))
    .fillna(method="ffill")
    .fillna(0)
    .groupby("sign")
    .apply(lambda x: x.n.to_list())
    .to_list()
)

# 官方解法2
df = df.sort_values('n', ignore_index=True)
bins = (
    df.assign(diff=df.n.diff(1))
    .query('diff>1')
    .n
    .to_list()
)

(
    df.set_axis(pd.cut(df.n, [0]+bins, right=False))
    .groupby(lambda x:x, dropna=False)
    .apply(lambda x: x.n.to_list())
    .to_list()
)
```

---

## [计算员工KPI完成率](https://gairuo.com/m/pandas-kpi-completion-rate)

> 题目：计算每位员工在每个月的**KPI完成率**（完成率 = 完成值 / 目标值 * 100%），并最终形成一个以`员工姓名`为索引，以月份为列，以完成率为值的清晰表格

```python
from io import StringIO

# 模拟的源数据字符串
data = """
员工姓名,一月目标,一月完成,二月目标,二月完成,三月目标,三月完成
张三,100,95,100,110,100,80
李四,150,140,150,160,150,150
王五,80,90,80,75,80,85
"""

# 读取数据
df = pd.read_csv(StringIO(data))

# 所需结果
"""
员工姓名     一月     二月         三月
张三     95.00%  110.00%   80.00%
李四     93.33%  106.67%  100.00%
王五    100.00%   93.75%  106.25%
"""
```

> 解法

```python
(
    df
    # 1. 使用melt将宽表转为长表，标识列为‘员工姓名’
    .melt(id_vars="员工姓名", var_name="月份指标", value_name="数值")
    # 2. 拆分‘月份指标’列：提取月份和指标类型
    .assign(
        # 提取前两个字符作为月份
        月份=lambda x: x["月份指标"].str[:2],
        # 提取剩余字符作为指标（目标/完成）
        指标=lambda x: x["月份指标"].str[2:],
    )
    # 3. 删除不再需要的原始复合列
    .drop(columns="月份指标")
    # 4. 构建透视表，行是员工和月份，列是指标，值是数值
    .pivot_table(
        index=["员工姓名", "月份"], columns="指标", values="数值"
    ).reset_index()
    # 5. 计算完成率
    .assign(完成率=lambda x: x["完成"] / x["目标"])
    # 6. 为了最终呈现，再次构建透视表：员工为行，月份为列，完成率为值
    .pivot_table(index="员工姓名", columns="月份", values="完成率")
    # 7. 将小数格式化为百分比字符串
    .map(lambda x: f"{x:.2%}")
)
```

---

## [将 Excel 按日期分列显示](https://gairuo.com/m/pandas-displays-excel-by-date)

> 题目：将一个平铺的数据，按日期显示，同一日期的产品则分列显示，不同产品合并显示在列上

```python
from io import StringIO

data = """
日期	产品	数据1	数据2	数据3	数据4	数据5	数据6
1月	产品1	1	2	3	4	5	6
1月	产品2	2	3	4	5	6	7
2月	产品1	1	2	3	4	5	6
2月	产品2	2	3	4	5	6	7
"""

df = pd.read_csv(StringIO(data),sep=r"\s+")

# 所需结果
'''
产品 产品1                     产品2
   数据1 数据2 数据3 数据4 数据5 数据6 数据1 数据2 数据3 数据4 数据5 数据6
日期
1月   1   2   3   4   5   6   2   3   4   5   6   7
2月   1   2   3   4   5   6   2   3   4   5   6   7
'''
```

> 解法

```python
# 官方解法
(df.groupby("日期").apply(lambda x: x.drop("日期", axis=1).set_index("产品").stack()))

# 个人解法
df.set_index(["日期","产品"]).stack().unstack([1,2])
```

---

## [按列标签包含值计算每行的和](https://gairuo.com/m/pandas-sum-row-column-labels)

> 题目：计算出每一行每个数字所在的列数字之和。如以上数据有 1 到 4 共 4 个数据，计算 2 所在的 `1_2 2_3 2_4` 这三列每行的和。最后将它们拼接在原数据右则

```python
import io

data = io.StringIO(
'''
1_2 1_3 1_4 2_3 2_4 3_4
1   5   2   8   2   2
4   3   4   5   8   5
8   8   8   9   3   3
4   3   4   4   8   3
8   0   7   4   2   2
''')

df = pd.read_csv(data, sep=r'\s+')

# 所需结果
'''
   1_2  1_3  1_4  2_3  2_4  3_4   1   2   3   4
0    1    5    2    8    2    2   8  11  15   6
1    4    3    4    5    8    5  11  17  13  17
2    8    8    8    9    3    3  24  20  20  14
3    4    3    4    4    8    3  11  16  10  15
4    8    0    7    4    2    2  15  14   6  11
'''
```

> 解法

```python
(
    df.join(
        df.T
        .assign(foo = lambda x:x.index.str.split("_"))
        .explode("foo")
        .groupby("foo")
        .sum()
        .T)
)
```

---

## [为连续重复值自动生成分组编号](https://gairuo.com/m/pandas-auto-increment-group-numbers)

> 题目：分配递增的编号，使得相同的连续值归为同一组，而一旦值变化，编号就 +1

```python
my_list = ['Apple', 'Apple', 'Orange', 'Orange','Orange','Banana']
df = pd.DataFrame(my_list, columns=['fruit'])

# 所需结果
'''
    fruit  label
0   Apple      1
1   Apple      1
2  Orange      2
3  Orange      2
4  Orange      2
5  Banana      3
'''
```

> 解法

```python
# 方法1:pd.factorize() 来生成枚举值，取到枚举数字部分
df.assign(label = pd.factorize(df["fruit"])[0] + 1)

# 方法2:按 fruit 进行分组，然后取每个组的编号
df.assign(label = df.groupby("fruit",sort=False).ngroup() + 1)

# 方法3:差异检测标记组起点，再累加生成分组ID，最终将连续相同值归类为同一数字标签
df.assign(label = df["fruit"].ne(df["fruit"].shift()).cumsum())
```

---

## [根据时间间隔和期数展开数据](https://gairuo.com/m/pandas-expands-intervals-periods)

> 题目：根据 interval_days 和 n_times 进行扩展，得到一个完成的时序数据

```python
import io

data = '''
n_times interval_days  date
3       4              03-01  
1       4              03-02
2       5              03-05
4       3              03-07
'''

df = pd.read_csv(io.StringIO(data), sep=r'\s+')

# 所需结果
'''
n_times  interval_days  date_range
      3              4  2024-03-01
      3              4  2024-03-05
      3              4  2024-03-09
      1              4  2024-03-02
      2              5  2024-03-05
      2              5  2024-03-10
      4              3  2024-03-07
      4              3  2024-03-10
      4              3  2024-03-13
      4              3  2024-03-16
'''
```

> 解法

```python
def func(ser: pd.Series) -> pd.DatetimeIndex:
    return pd.date_range(
        start=f'2025-{ser.date}',
        periods=ser.n_times,
        freq=f'{ser.interval_days}D'
    )

(
    df.assign(date_range=df.apply(func, axis=1))
    .explode('date_range')
)
```

---

## [元素出现最早的行号](https://gairuo.com/m/pandas-lines-earliest-appearance)

> 题目：增加一列，对应值为 data 中所有元素第一次出现在表中最早的行数

```python
import io


data = (['a', 's'],
        ['b'],
        ['c','b', 's'], 
        ['c', 'c', 't'], 
        ['t']
       )

df = pd.DataFrame({'data': data})

# 所需结果
'''
        data  foo
0     [a, s]    1
1        [b]    2
2  [c, b, s]    1
3  [c, c, t]    3
4        [t]    4
'''
```

> 解法：
>
> - 先对 data 列进行爆炸，这样所展开了所有的元素，注意，索引是此元素所在的行索引。再删除掉重复的元素，就只剩下所有的不重复的值，索引加 1 就是我们说的元素首次出现的序号
>
> - 再构造映射表（字典），键为元素，值为首次出现的行序号
>
> - 再用`map()`迭代原始 df 的 data 列，用推导式逐个利用映射表计算首次出现的行号，再取最小值，这样就得到想要的数据

```python
ser = df.data.explode().drop_duplicates().rename(lambda x: x + 1)

_map = {v: i for i, v in ser.items()}

no = df.data.map(lambda x: min(_map.get(i) for i in x))

df.assign(foo=no)
```

---

## [扩展逗号分隔字符串数据](https://gairuo.com/m/pandas-extension-comma-separated-string)

> 题目：将 c1 按逗号拆分，爆炸为一行一行数据，然后将 c1 后边有逗号的扩展成行，没逗号的只写在第一行

```python
import io

data = """
a  b         c1   c2     c3  c4
A  W      1,2,3  1,2  1,3,3  g
B  X  1,2,3,4,5    k  1,4,7  h
"""

df = pd.read_csv(io.StringIO(data), sep=r"\s+")
# 所需结果
'''
   a  b c1 c2 c3  c4
0  A  W  1  1  1  g
1  A  W  2  2  3   
2  A  W  3     3   
3  B  X  1  k  1  h
4  B  X  2     4   
5  B  X  3     7   
6  B  X  4         
7  B  X  5         
'''
```

> 解法

```python
# 个人解法
(
    df.set_index(["a","b"])
    .map(lambda x:x.split(",") if "," in x else [x])
    .assign(max_len = lambda x:x.c1.map(len))
    .apply(lambda x:x.loc["c1":"c4"].map(lambda y:y + ([""] * (x["max_len"]-len(y)))),axis=1)
    .explode(["c1","c2","c3","c4"])
    .reset_index()
)

# 官方解法
def func(s: pd.Series):
    first = s.iloc[0]
    if s.name == 'c1':
        return s
    elif isinstance(first, list):
        space_len = len(s) - len(first)
        values = first + ['']*space_len
        return values
    else:
        values = [first] + ['']*(len(s)-1)
        return values

(
    df.map(lambda x: x.split(',') if ',' in x else x)
    .explode('c1')
    .set_index(['a', 'b'])
    .groupby(level=[0, 1])
    .transform(func)
    .reset_index()
)
```

---

## [根据是否包含扩展数据](https://gairuo.com/m/pandas-includes-extended-data)

> 题目：将第一个表中的 A，全部跟第二表中的B全部模糊匹配以下（只要B中包含A都显示出来），然后将 A 在 B 中包含的全部提取出来

```python
data1 = {'A': ['ABC', 'EF', 'DF', 'OP']}
data2 = {'B': ['ABC123', 'AB', 'ABCEF', 'DFWAQ', 'ABCEF', 'ABCEFBC']}
df1 = pd.DataFrame(data1)
df2 = pd.DataFrame(data2)

# 所需结果
'''
A        B
ABC   ABC123
ABC    ABCEF
ABC    ABCEF
ABC  ABCEFBC
 EF    ABCEF
 EF    ABCEF
 EF  ABCEFBC
 DF    DFWAQ
'''
```

> 解法

```python
(
    df1.assign(B = df1["A"].map(lambda x: df2["B"][df2["B"].str.contains(x)]))
    .explode("B")
    .dropna()
)
```

---

## [组合列值并扩展为多行](https://gairuo.com/m/pandas-combines-column-expands-rows)

> 题目：id 列内容不变，增加 foo 列，foo 的内容由 A 列开头与后续其他列组合而成，不同列值之间用逗号隔开

```python
data = {'id': ['i1', 'i2', 'i3'],
        'A': ['a1', 'a2', 'a3'],
        'C': ['', '', 'c3',],
        'D': ['d1', 'd2', 'd3'],
        'E': ['', 'e2', 'e3']}

df = pd.DataFrame(data)

# 所需结果
'''
   id          foo
0  i1        a1,d1
1  i2        a2,d2
1  i2        a2,e2
1  i2     a2,d2,e2
2  i3        a3,c3
2  i3        a3,d3
2  i3        a3,e3
2  i3     a3,c3,d3
2  i3     a3,c3,e3
2  i3     a3,d3,e3
2  i3  a3,c3,d3,e3
'''
```

> 解法

```python
import itertools

def func(ser: pd.Series):
    head = ser.A
	# 两个 Series 长度不同是因为标签对齐机制
    part = ser.loc['C':].loc[~(ser == '')]
    res = []
    
    for n in range(1, len(part)+1):
        it = itertools.combinations(part, n)
        res += it

    s = [','.join([ser.A]+[*i]) for i in res]
    
    return s

(
    df[['id']].assign(foo=df.apply(func, axis=1))
    .explode('foo')
)
```

---