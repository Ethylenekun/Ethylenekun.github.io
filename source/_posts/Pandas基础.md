---
title: Pandas基础
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas基础
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 6
abbrlink: 488d07c0
---
# Pandas教程

##  基础功能

### Excel对象

1. [pd.ExcelFile](https://gairuo.com/p/pandas-excel-file)
   - 属性
     - `sheet_names`：返回 Excel 文件中所有工作表的名称列表
     - `book`：与 openpyxl 或 xlrd 等 Excel 处理引擎交互，允许用户访问 Excel 文件的底层对象
       - `book.worksheets[0].max_row`：返回第一个工作表的最大行数
   - 方法
     - `parse`：读取指定工作表的数据并返回为 Pandas DataFrame。可以指定参数，如 `header`、`skiprows`、`usecols` 等，以控制读取行为
     - `close`：关闭excel文件，释放资源
2. [pd.ExcelWriter](https://gairuo.com/p/pandas-excel-writer)
   - 参数
     - `mode`：默认值为`w`
       - `w`: 写入模式，若文件存在则覆盖
       - `a`: 追加模式，向现有文件中添加数据
     - `if_sheet_exists`：默认值为`error`
       - `error`: 抛出错误，默认行为
       - `new`: 创建一个新的工作表
       - `replace`: 替换已存在的工作表
       - `overlay`:在已存在的工作表基础上追加

> 将多个 DataFrame 写入同一个工作表中的不同区域

```python
data1 = {
    '姓名': ['艾丽丝', '鲍勃', '查理'],
    '年龄': [30, 25, 35],
    '城市': ['纽约', '洛杉矶', '芝加哥']
}
df1 = pd.DataFrame(data1)

data2 = {
    '产品': ['A', 'B', 'C'],
    '销量': [1000, 800, 600],
    '收入': [50000, 40000, 30000]
}
df2 = pd.DataFrame(data2)

with pd.ExcelWriter('output_multiple_regions.xlsx', engine='xlsxwriter') as writer:
    # 将第一个 DataFrame 写入 'Sheet1' 的 A1 单元格
    df1.to_excel(writer, sheet_name='Sheet1', startrow=0, startcol=0, index=False)
    
    # 将第二个 DataFrame 写入 'Sheet1' 的 E1 单元格
    df2.to_excel(writer, sheet_name='Sheet1', startrow=0, startcol=4, index=False)
```

> 追加数据至已存在的工作表

```python
with pd.ExcelWriter(
    "data/demo.xlsx",
    mode="a",
    engine="openpyxl", # 默认引擎会报错
    if_sheet_exists="overlay"
) as file:
    
    df.to_excel(file,
                sheet_name="demo",
                startrow=pd.ExcelFile("data/demo.xlsx").book.worksheets[0].max_row, # 获取最大行数
                header=False, # 追加取消表头
                index=False
                )
```

---

## 函数应用

### [pipe()管道方法](https://gairuo.com/p/pandas-pipe)

```python
# 传入位置参数或关键字参数
func(g(h(df), arg1=a), arg2=b, arg3=c)

# 等同于下方写法
(	
    df.pipe(h)
    .pipe(g, arg1=a)
    .pipe(func, arg2=b, arg3=c)
)
```

> 函数 fun 的第一个参数不是数据本身，可采用一元组来传递

```python
# 数据是第二个参数
def fun(x, my_df):
    return my_df*x

# 调用和传值方法
df.pipe((fun, 'my_df'), 2)
df.pipe((fun, 'my_df'), x=2)
```

### [map()方法](https://gairuo.com/p/pandas-map)

### [apply()方法](https://gairuo.com/p/pandas-apply)

- `result_type=None`：`{‘expand’, ‘reduce’, ‘broadcast’, None}`, 默认 None，仅在 axis=1 (columns) 时都起作用：
    - `expand` : 类似列表的结果将转换为列
    - `reduce` : 如果可能，返回一个序列，而不是像列表一样展开结果。这与 `expand`相反
    - `broadcast` : 结果将被广播到数据帧的原始形状，原始索引和列将被保留
    - `None`：默认行为（None）取决于应用函数的返回值：类似列表的结果将作为一Series 结果返回。但是，如果 apply 函数返回一个 Series，这些序列将展开为列

```python
s = pd.Series([20, 21, 12], index=["London", "New York", "Helsinki"])

def subtract_custom_value(x,v1,v2,v3):
    return x - v1 - v2 + v3

s.apply(subtract_custom_value, args=(5,10),v3=100)
```

### [agg()方法](https://gairuo.com/p/pandas-agg)

-  聚合滚动窗口 `Rolling.agg`

```python
df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6], "C": [7, 8, 9]})

df.rolling(2).agg({"A":"sum","B":"min"})
'''
     A    B
0  NaN  NaN
1  3.0  4.0
2  5.0  5.0
'''
```

### [transform方法](https://gairuo.com/p/pandas-transform)

[按组显示列值的列表](https://gairuo.com/m/pandas-display-column-lists-group)

> 题目：需要增加一个名为「所属组别的值」的列，对应值为当前组别下「值」的列表

```python
from io import StringIO
import pandas as pd

data = '''
组别  值
A   1
A   2
A   3
B   4
B   5
C   6
C   7
B   8
'''

df = pd.read_csv(StringIO(data), sep=r'\s+')

# 所需结果
'''
  组别  值 所属组别的值
0  A  1  1,2,3
1  A  2  1,2,3
2  A  3  1,2,3
3  B  4  4,5,8
4  B  5  4,5,8
5  C  6    6,7
6  C  7    6,7
7  B  8  4,5,8
'''
```

> 解法

```python
foo = df.groupby('组别').值.transform(lambda x: [x.to_list()] * len(x))
df.assign(所属组别的值=foo)
```

---

## 多级索引

> 性能问题：传入元组列表或单个元组或返回前二者的函数时，需要先进行索引排序以避免性能警告

```python
df.set_index(["team","name"],inplace=True)

df.loc[("E","Liver")] # 若数据过多,会提示PerformanceWarning: indexing past lexsort depth may impact performance.

# 优先进行排序后索引
df.sort_index().loc[("E","Liver")]

# 多级索引切片,若未排序索引,不论切片的端点元素是否唯一,都会报错UnsortedIndexError
df.loc[("E",8):] # 不唯一端点		
df.loc[("E",97):] # 唯一端点

df.sort_index().loc[("E",8):] # 不会报错
```

> 特殊写法：可以对多层的元素在交叉组合后进行索引，但同时需要指定loc 索引器的列，全选则用`:`表示

```python
df.set_index(["team","Q1"],inplace=True)

# 索引为列表组成的笛卡尔积的组合
df.loc[(["A","B"],[57,9]),:] # :或者明确的列名是必须的 / 参数为列表嵌套在元组内

# 索引为元祖内的组合
df.loc[[("A",9),("B",57)]] # 参数为元祖嵌套在列表内
```

> 题目：sample()函数中主要的参数为n、axis、frac、replace 和weights，前3 个分别指抽样数量、抽样的方向（0 为行、1 为列）和抽样比例（如0.3 表示从总体中抽出30%的样本）。replace 和weights分别指是否放回和每个样本抽样的相对概率，replace 为True 表示有放回抽样

```python
# 使用iloc和np.random.choice实现sample函数
def sample(df:pd.DataFrame,n=None,replace=None,axis=None,weights=None,frac=None):
  temp_df = df.copy()

  if n != None and frac != None:
    raise ValueError("n和frac只能存在一个")
  
  if n == None:
    n = int(len(temp_df) * frac)

  if isinstance(weights,list):
    weights = np.array(weights)

  if not isinstance(weights,np.ndarray):
    weights = np.ones(df.shape[axis]) / df.shape[axis]
    # weights = np.repeat(1 / df.shape[axis],df.shape[axis]) 相同效果
  
  idx = np.random.choice(range(df.shape[axis]),size=n,replace=replace,p=weights/weights.sum())

  return temp_df.iloc[idx,:] if axis == 0 else temp_df.iloc[:,idx]
```

## 数据合并

### [时序数据合并](https://gairuo.com/p/pandas-timeseries-merging)

`pd.merge_asof`：类似于`left-join`

- `direction`
  - **默认** `backward`向后，搜索选择右数据中`on`键小于或等于左键的最后一行，比它小的里边最大的
  - `forward`向前，搜索选择右侧数据中的第一行，其`on`键大于或等于左侧的键，比它大的里边最大的
  - `nearest`最近，搜索选择右侧数据中的行，该行的`on`键在绝对距离上最接近左侧的键，和它差值绝对值最小的
- `allow_exact_matches`
  - 如为 True（默认）, 允许与相同的值匹配（即小于或等于/大于或等于）
  - 如为 False, 不匹配相同的值（即严格要求小于/大于，就是不能等于）

```python
left = pd.DataFrame({"a": [1, 5, 10], "left_val": ["a", "b", "c"]})

right = pd.DataFrame({"a": [1, 2, 3, 6, 7], "right_val": [1, 2, 3, 6, 7]})

pd.merge_asof(left, right, on="a", direction="forward")
"""
    a left_val  right_val
0   1        a        1.0
1   5        b        6.0
2  10        c        NaN
"""
```

### [逐元素合并](https://gairuo.com/p/pandas-combines)

`df.combine_first`

```python
df1 = pd.DataFrame({'A': [None, 0], 'B': [4, None]})
df2 = pd.DataFrame({'B': [3, 3], 'C': [1, 1]}, index=[1, 2])
df1.combine_first(df2)
'''
     A    B    C
0  NaN  4.0  NaN
1  0.0  3.0  1.0
2  NaN  3.0  1.0
'''
```

`df.update`

> 使用`pd.Series`更新，必须设置其`name`属性对应列

```python
df = pd.DataFrame({'A': ['a', 'b', 'c'],
                   'B': ['x', 'y', 'z']})
new_column = pd.Series(['d', 'e'], name='B', index=[0, 2])
df.update(new_column)	
df
'''
   A  B
0  a  d
1  b  y
2  c  e
'''
```

> 其他包含NaN，则相应的值不会在原始数据帧中更新

```python
df = pd.DataFrame({'A': [1, 2, 3],
                   'B': [400, 500, 600]})
new_df = pd.DataFrame({'B': [4, np.nan, 6]})
df.update(new_df)
df
'''
   A      B
0  1    4.0
1  2  500.0
2  3    6.0
'''
```

### [数据对比](https://gairuo.com/p/pandas-compare)

> `df.compare `

```python
df = pd.DataFrame(
    {
        "col1": ["a", "a", "b", "b", "a"],
        "col2": [1.0, 2.0, 3.0, np.nan, 5.0],
        "col3": [1.0, 2.0, 3.0, 4.0, 5.0],
    },
    columns=["col1", "col2", "col3"],
)

df2 = df.copy()
df2.loc[0, "col1"] = "c"
df2.loc[2, "col3"] = 4.0

# 将差异堆叠在行上：
df.compare(df2, align_axis=0)
'''
        col1  col3
0 self     a   NaN
  other    c   NaN
2 self   NaN   3.0
  other  NaN   4.0
'''


# 保留相等的值
df.compare(df2, keep_equal=True)
'''
  col1       col3
  self other self other
0    a     c  1.0   1.0
2    b     b  3.0   4.0
'''


# 保留所有原始行和列
df.compare(df2, keep_shape=True)
'''
  col1       col2       col3
  self other self other self other
0    a     c  NaN   NaN  NaN   NaN
1  NaN   NaN  NaN   NaN  NaN   NaN
2  NaN   NaN  NaN   NaN  3.0   4.0
3  NaN   NaN  NaN   NaN  NaN   NaN
4  NaN   NaN  NaN   NaN  NaN   NaN
'''
```

> `df.align`

```python
df1 = pd.DataFrame([[1,2,3,4], [6,7,8,9]], 
                   columns=['D', 'B', 'E', 'A'],
                   index=[1,2])

df2 = pd.DataFrame([[10,20,30,40], [60,70,80,90], [600,700,800,900]],
                   columns=['A', 'B', 'C', 'D'],
                   index=[2,3,4])

a1, a2 = df1.align(df2, join='outer', axis=1)
print(a1)
print(a2)
'''
   A  B   C  D  E
1  4  2 NaN  1  3
2  9  7 NaN  6  8

A    B    C    D   E
2   10   20   30   40 NaN
3   60   70   80   90 NaN
4  600  700  800  900 NaN
'''
```

---

## 数据清洗

### 扩展：np.nan和pd.NA

| **特性**       | **np.nan**                                      | **pd.NA**                                      |
| -------------- | ----------------------------------------------- | ---------------------------------------------- |
| **数据类型**   | 仅支持 **float** 类型（会强制转换）             | 支持 **Nullable 类型**（Int64/boolean/string） |
| **类型一致性** | 整数列插入 `np.nan` 会变为 float                | 保持原类型（如 Int64 列仍为整数）              |
| **相等性判断** | `np.nan == np.nan` → `False`                    | `pd.NA == pd.NA` → `pd.NA`（需用 `isna()`）    |
| **逻辑运算**   | 遵循 numpy 规则（如 `np.nan & True` → `False`） | 遵循三值逻辑（`pd.NA & True` → `pd.NA`）       |
| **适用场景**   | 兼容 numpy 旧代码                               | pandas 专属，推荐用于新数据类型                |

> `Nullable`类型：解决传统 NumPy 数据类型在处理缺失值时的局限性而设计的一套**专属数据类型**。它允许整数、布尔值等类型直接包含缺失值（`pd.NA`），同时**保持数据类型的一致性**，避免了传统类型的 “强制降级” 问题

| **数据类型** | **Nullable 类型**       | **说明**                                 |
| ------------ | ----------------------- | ---------------------------------------- |
| 整数         | `Int64`/`Int32`/`Int16` | 可空整数（大写 `I`），避免转为 float     |
| 布尔值       | `boolean`               | 可空布尔（`True`/`False`/`pd.NA`）       |
| 字符串       | `string`                | 可空字符串（替代 `object` 类型，更高效） |
| 浮点数       | `Float64`/`Float32`     | 可空浮点数（虽不常用，但语义更明确）     |

```python
# 算数运算的特殊情况
1 * pd.NA, 1 ** pd.NA, pd.NA ** 0  # (<NA>, 1, 1)

# 逻辑运算
True | pd.NA, False | pd.NA  # (True, <NA>)
```

### [数据替换](https://gairuo.com/p/pandas-replace)

> `replace`

```python
ser = pd.Series(range(5))

# 将0替换为5
ser.replace(0,5)

# 对应替换
ser.replace([0, 1, 2, 3, 4], [4, 3, 2, 1, 0])
ser.replace([0, 1, 2, 3, 4], 10)

# 字典替换
ser.replace({0: 10, 1: 100})

# 将 a 列的 0 b 列中的 5 替换为 100
df.replace({'a': 0, 'b': 5}, 100)

#  指定列里的替换规划
df.replace({'a': {0: 100, 4: 400}})
```

> 正则替换

```python
d = {'a': list(range(4)),
     'b': list('ab..'),
     'c': ['a', 'b', np.nan, 'd']
    }
df = pd.DataFrame(d)

df.replace([r'\.', r'(a)'], ['dot', r'\1stuff'], regex=True)

df.replace(regex={"b": {r"\s*\.\s*": np.nan}})

df.replace([r'\s*\.\s*', r'a|b'], np.nan, regex=True)
```

- `case_when`

> [case_when教程](https://gairuo.com/p/pandas-case-when)

```python
# caselist为元祖嵌套的列表,元祖内为(判定条件产生的布尔序列,替换值)
df.assign(level = df["Q1"].case_when([
  (df["Q1"].lt(60),"E"),
  (df["Q1"].lt(70),"D"),
  (df["Q1"].lt(80),"C"),
  (df["Q1"].lt(90),"B"),
  (df["Q1"].eq(df["Q1"]),"A") # 构造一个等长的值全为 True 的布尔序列实现else效果
]))
```

---

## 分组聚合

### gb.apply

- 单列应用`pd.Series`

```python
# Series的索引会添加到分组的索引中
df.groupby("team")['name'].apply(lambda x:pd.Series([1,2]))
'''
team   
A     0    1
      1    2
B     0    1
      1    2
C     0    1
      1    2
D     0    1
      1    2
E     0    1
      1    2
Name: name, dtype: int64
'''
```

- 单列/多列应用`pd.DataFrame`

```python
# 单列/多列效果相同
df_ = pd.DataFrame(np.ones([2,2]),index=[*"AB"],columns=[*"ab"])
df.groupby("team").apply(lambda x:df_)
'''
		a	b
team			
A	A	1.0	1.0
	B	1.0	1.0
B	A	1.0	1.0
	B	1.0	1.0
C	A	1.0	1.0
	B	1.0	1.0
D	A	1.0	1.0
	B	1.0	1.0
E	A	1.0	1.0
	B	1.0	1.0
'''
```

- 多列应用`pd.Series`

```python
# Series的索引会成为columns
df.groupby("team").apply(lambda x:pd.Series([1,2]))
'''
	0	1
team		
A	1	2
B	1	2
C	1	2
D	1	2
E	1	2
'''
```

### [pd.Grouper() 分组器](https://gairuo.com/p/pandas-grouper)

- **closed** (`'left'` 或 `'right'`)：
  在时间序列分组时，指定区间是左闭还是右闭。如果未指定 `freq` 参数，此参数无效。
- **label** (`'left'` 或 `'right'`)：
  指定区间的标记方式，`'right'` 表示右边界，`'left'` 表示左边界。仅在 `freq` 参数指定时有效。

- **origin** (`Timestamp` 或 `str`, 默认为 `'start_day'`)：
  调整分组的起点时间戳。可选值包括：
  - `'epoch'`：从 `1970-01-01 `开始
  - `'start'`：时间序列的第一个值
  - `'start_day'`：时间序列的第一个午夜
  - `'end'`：时间序列的最后一个值
  - `'end_day'`：时间序列的最后一天的午夜

### [数据分箱](https://gairuo.com/p/pandas-data-binning)

> [pd.cut](https://gairuo.com/p/pandas-cut)

- **`x`**：需要分箱的一维数据（如 `Series`、`ndarray`）。
- **`bins`**：
  - 如果是整数，表示将数据分成指定数量的等宽区间 -> **类似np.linspace形成等差区间**
  - 如果是序列，表示自定义的区间边界。
- **`right`**：是否为右闭区间，默认为 `True`（区间形如 `(a, b]`）。
- **`labels`**：
  - 指定每个区间的标签（如 `['Low', 'Medium', 'High']`）。
  - 如果为 `False`，则返回区间编号而不是标签。
- **`retbins`**：是否返回实际使用的分箱边界，默认 `False`。
- **`precision`**：区间端点的小数精度，默认为 3。
- **`include_lowest`**：是否将第一个区间的左端点包含在内（仅在 `right=True` 时生效）。
- **`duplicates：`**
  - 如果分箱边界有重复值，默认抛出异常。
  - 可设置为 `'drop'` 忽略重复边界。

```python
# 语法
pd.cut(x, bins, right=True,
       labels=None, retbins=False,
       precision=3, include_lowest=False,
       duplicates='raise')
```

> [pd.qcut](https://gairuo.com/p/pandas-qcut)

1. **`x`**（必选）
   - 一维数据，如 `list`、`ndarray`、`Series` 等。
   - 是需要分箱的数据。
2. **`q`**（必选）
   - 分位数的数量或具体的分位点。
   - 如果是整数，则表示分成 `q` 个等分位区间（例如，`q=4` 分成四分位数）。
   - 如果是列表，则表示指定分位点（如 `[0, 0.25, 0.5, 0.75, 1]`）。
3. **`labels`**（可选，默认为 `None`）
   - 为每个区间指定标签。
   - 可以是布尔值、字符串列表或整数列表：
     - 如果为 `False`，返回区间编号（从 0 开始）。
     - 如果为 `None`，返回默认的区间（如 `(a, b]` 格式）。
     - 如果提供自定义标签，数量必须与区间数量一致。
4. **`retbins`**（可选，默认为 `False`）
   - 是否返回实际使用的分箱边界。
   - 如果为 `True`，则返回分箱结果和分箱边界。
5. **`precision`**（可选，默认为 `3`）
   - 分箱边界的精度，即小数点后的位数。
6. **`duplicates`**（可选，默认为 `'raise'`）
   - 如果分箱边界中有重复值的处理方式：
     - `'raise'`：抛出异常。
     - `'drop'`：忽略重复边界。

```python
# 语法
pandas.qcut(
    x, 
    q, 
    labels=None, 
    retbins=False, 
    precision=3, 
    duplicates='raise'
)
```

---

## 重塑透视

### [数据透视](https://gairuo.com/p/pandas-pivoting)

> `df.pivot`

- index：新 df 的索引列，用于分组，如果为None，则使用现有索引
- columns：新 df 的列，如果透视后有重复值会报错
- values：用于填充 df 的列。 如果未指定，将使用所有剩余的列，并且结果将具有按层次结构索引的列

```python
df = pd.DataFrame({'foo': ['one', 'one', 'one', 'two', 'two',
                           'two'],
                   'bar': ['A', 'B', 'C', 'A', 'B', 'C'],
                   'baz': [1, 2, 3, 4, 5, 6],
                   'zoo': ['x', 'y', 'z', 'q', 'w', 't']})

df.pivot(index='foo', columns='bar', values='baz')
'''
bar  A   B   C
foo
one  1   2   3
two  4   5   6
'''

# 多层索引，取其中一列
df.pivot(index='foo', columns='bar')['baz']
'''
bar  A   B   C
foo
one  1   2   3
two  4   5   6
'''
```

> `df.pivot_table`

- values: 要聚合的列或者多个列
- index: 在数据透视表索引上进行分组的键
- columns: 在数据透视表列上进行分组的键
- aggfunc: 用于聚合的函数, 默认是 numpy.mean
- margins:  给行和列添加一个汇总，默认名字是ALL
- margins_name: 替换默认汇总行和列的名字

```python
df = pd.DataFrame({"A": ["foo", "foo", "foo", "foo", "foo",
                         "bar", "bar", "bar", "bar"],
                   "B": ["one", "one", "one", "two", "two",
                         "one", "one", "two", "two"],
                   "C": ["small", "large", "large", "small",
                         "small", "large", "small", "small",
                         "large"],
                   "D": [1, 2, 2, 3, 3, 4, 5, 6, 7],
                   "E": [2, 4, 5, 5, 6, 6, 8, 9, 9]})
df
'''
     A    B      C  D  E
0  foo  one  small  1  2
1  foo  one  large  2  4
2  foo  one  large  2  5
3  foo  two  small  3  5
4  foo  two  small  3  6
5  bar  one  large  4  6
6  bar  one  small  5  8
7  bar  two  small  6  9
8  bar  two  large  7  9
'''

# 不同值使用不同的聚合计算方式
pd.pivot_table(
    df,
    values=["D", "E"],
    index=["A", "C"],
    aggfunc={"D": "mean", "E": ["min", "max", "mean"]},
)

# 汇总边际，给列的每层加一个 all 列进行汇总，计算方式与 aggfunc 相同
pd.pivot_table(df, values='D', index=['A', 'B'],
               columns=['C'],  aggfunc=np.sum,
               margins=True)
```

### [交叉表 crosstab](https://gairuo.com/p/pandas-crosstab)

- index：类数组，在行中按分组的值。
- columns：类数组的值，用于在列中进行分组。
- values：类数组的，可选的，要根据因素汇总的值数组。
- aggfunc：函数，可选，如果未传递任何值数组，则计算频率表。
- rownames：序列，默认为None，必须与传递的行数组数匹配。
- colnames：序列，默认值为None，如果传递，则必须与传递的列数组数匹配。
- margins：布尔值，默认为False，添加行/列边距（小计）
- normalize：布尔值，{'all'，'index'，'columns'}或{0,1}，默认为False。 通过将所有值除以值的总和进行归一化。

```python
# 语法
pd.crosstab(index, columns, values=None, rownames=None,
colnames=None, aggfunc=None, margins=False,
margins_name: str = 'All', dropna: bool = True,
normalize=False) -> 'DataFrame'
```

```python
# 实例:A B 两列进行交叉，A有不重复的值1和2，B有3和4。交叉后组成了新的数据，具体的值为对应行列上的组合在原数据中的数量
df = pd.DataFrame({'A': [1, 2, 2, 2, 2],
                   'B': [3, 3, 4, 4, 4],
                   'C': [1, 1, np.nan, 1, 1]})

pd.crosstab(df['A'], df['B'],
            values=df['C'],
            aggfunc=np.sum,
            normalize=True,
            margins=True)

'''
B       3    4   All
A
1    0.25  0.0  0.25
2    0.25  0.5  0.75
All  0.50  0.5  1.00
'''
```

### [逆透视melt](https://gairuo.com/p/pandas-melt)

- id_vars: tuple，list或ndarray（可选），用作标识变量的列。
- value_vars: tuple，列表或ndarray，可选，要取消透视的列。 如果未指定，则使用未设置为id_vars的所有列。
- var_name: scalar，用于“变量”列的名称。 如果为None，则使用frame.columns.name或“variable”。
- value_name: scalar，默认为“ value”，用于“ value”列的名称。
- col_level: int或str，可选，如果列是MultiIndex，则使用此级别来融化。

### [虚拟变量/哑变量](https://gairuo.com/p/pandas-dummies)

`pd.get_dummies()` 是将一个或者多个列的去重值做为新表的列，每个列的值由0和1组成，在原来此位为此列名的值为1，不是的为0，这样就形成了一个由 0 和1 组成的特征矩阵

```python
df = pd.DataFrame({'key': list('bbacab'), 'data1': range(6)})

pd.get_dummies(df['key'], prefix='key')
'''
   key_a  key_b  key_c
0      0      1      0
1      0      1      0
2      1      0      0
3      0      0      1
4      1      0      0
5      0      1      0
'''

df = pd.DataFrame({'A': ['a', 'b', 'a'],
                   'B': ['c', 'c', 'b'],
                   'C': [1, 2, 3]})
pd.get_dummies(df, columns=['A'])
'''
   B  C  A_a  A_b
0  c  1    1    0
1  c  2    0    1
2  b  3    1    0
'''
```

### [factorize() 因子化值](https://gairuo.com/p/pandas-factorizing)

```python
codes, uniques = pd.factorize(['b', 'b', 'a', 'c', 'b'], sort=True)
codes
# array([1, 1, 0, 2, 1])
uniques
# array(['a', 'b', 'c'], dtype=object)


# 缺失值不会出现在唯一值列表中，在编码中将为 -1
codes, uniques = pd.factorize(['b', None, 'a', 'c', 'b'])
codes
# array([ 0, -1,  1,  2,  0])
uniques
# array(['b', 'a', 'c'], dtype=object)
```

---

