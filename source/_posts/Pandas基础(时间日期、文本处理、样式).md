---
title: Pandas基础(时间日期、文本处理、样式)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas基础(时间日期、文本处理、样式)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 6
abbrlink: 4cbd3882
---
# 时间日期

## dt对象

常用属性

- date、time、year、month、day、hour、minute、second、quarter
- dayofweek、dayofyear
- daysinmonth：返回对应月的天数
- month_name()、day_name()：返回月份和周几的英文名
- is_month_end、is_month_start、is_year_start、is_year_end、is_leap_year(判断是否是闰年)

操作

- round、ceil、floor
- strftime：对时间格式化
- isocalendar().week：返回时间序列的周数

```python
s = pd.Series(pd.date_range('2021-1-1 20:35:00',
'2021-1-1 22:35:00',
freq='45min'))

s.dt.ceil(freq="1h")
'''
0   2021-01-01 21:00:00
1   2021-01-01 22:00:00
2   2021-01-01 23:00:00
dtype: datetime64[ns]
'''
```

## [时间序列](https://gairuo.com/p/pandas-time-type)

> date_range

```python
pd.date_range("20210101", "20210201", freq="WOM-1MON")  # 每月第一个周一
```

[pd.period_range](https://gairuo.com/p/pandas-period-range)

```python
p_month = pd.Period("2024-01",freq="M")
print("Period对象：", p_month)  # 输出：2024-01
print("该时间段的开始时间：", p_month.start_time)  # 2024-01-01 00:00:00
print("该时间段的结束时间：", p_month.end_time)  # 2024-01-31 23:59:59.999999999

pd.period_range(start='2023Q1', end='2024Q4', freq='Q')

# PeriodIndex(['2023Q1', '2023Q2', '2023Q3', '2023Q4', '2024Q1', '2024Q2','2024Q3', '2024Q4'],dtype='period[Q-DEC]')
```

## [时间戳](https://gairuo.com/p/pandas-timestamp)

`pd.Timestamp`

```python
# 使用 python 的 datetime 库
import datetime
pd.Timestamp(datetime.datetime(2026,1,1,12,0,0))

# 指定时间字符串
pd.Timestamp("2025-01-01T12:30:00")

# 指定时间位置数字
pd.Timestamp(2026,1,1,12,0,0)

# 解析时间戳
pd.Timestamp(1513393355.5, unit="s")

# 特殊时间
pd.Timestamp("now")
pd.Timestamp('today').date() # 只取日期
pd.Timestamp("now").replace(year=2025,day=1) # 替换年月


# 获取时间戳的时间单位
ts = pd.Timestamp("now")

"ts 的时刻是{y}年{m}月{d}日{h}点{min}分{s}秒".format(
    y=ts.year, m=ts.month, d=ts.day, h=ts.hour, min=ts.minute, s=ts.second
) # 'ts 的时刻是2026年2月11日16点19分35秒'

# Timestamp 对象上定义一个value 属性，其返回的整数值代表从1970年1月1日0点到给定时间戳之间的纳秒数
ts.value

ts.strftime("%Y-%m-%d")
```

## [时间戳序列](https://gairuo.com/p/pandas-converting-timestamps)

1. `pd.to_datetime`

```python
# 指定数据格式
pd.to_datetime("20000101", format=r"%Y%m%d", errors="ignore") # Timestamp('2000-01-01 00:00:00')

# 指定数据单位
pd.to_datetime(1490195805, unit="s")  # Timestamp('2017-03-22 15:16:45')
```

> Pandas3.0中`pd.to_datetime`变化

```python
pd.to_datetime(pd.Series(["Jul 31, 2009", "2010-01-10", None])) # 3.0中会发生报错,需以下方方法转换

pd.Series(["Jul 31, 2009", "2010-01-10", None]).astype("datetime64[ns]")

pd.Series(["Jul 31, 2009", "2010-01-10", None]).map(pd.Timestamp)
```

> **时间日期格式化符号**

- %y 两位数的年份表示（00-99）
- %Y 四位数的年份表示（000-9999）
- %m 月份（01-12）
- %d 月内中的一天（0-31）
- %H 24小时制小时数（0-23）
- %I 12小时制小时数（01-12）
- %M 分钟数（00-59）
- %S 秒（00-59）

2. `pd.date_range`

```python
pd.date_range('2021-9-1','2021-9-21', freq='10D')
'''
DatetimeIndex(
['2021-09-01', '2021-09-11', '2021-09-21'],
dtype='datetime64[ns]', freq='10D')
'''

pd.date_range('2021-9-1', '2021-10-28', periods=6)
'''
DatetimeIndex(['2021-09-01 00:00:00', '2021-09-12 09:36:00',
'2021-09-23 19:12:00', '2021-10-05 04:48:00',
'2021-10-16 14:24:00', '2021-10-28 00:00:00'],
dtype='datetime64[ns]', freq=None)
'''
```

## [时间序列索引](https://gairuo.com/p/pandas-datetime-index)

### 索引访问

```python
rng = pd.date_range('1/1/2021', '12/1/2021', freq='BM')
ts = pd.Series(np.random.randn(len(rng)), index=rng)

ts['2021'] # 查询整个2021年的
ts['2021-6'] # 查询 2021年6月的
ts['2021-6':'2021-10'] # 6月到10月的
dft['2013-1':'2013-2-28 00:00:00'] # 精确时间
dft['2013-1-15':'2013-1-15 12:30:00']
dft2.loc['2013-01-05']

# 索引选择器
idx = pd.IndexSlice
dft2.loc[idx[:, '2013-01-05'], :]
# 带时区，原数据时区可能不是这个
df['2019-01-01 12:00:00+04:00':'2019-01-01 13:00:00+04:00']
```

### 截断索引

```python
rng2 = pd.date_range('2011-01-01', '2012-01-01', freq='W')
ts2 = pd.Series(np.random.randn(len(rng2)), index=rng2)

ts2.truncate(before='2011-11', after='2011-12')
'''
2011-11-06    0.437823
2011-11-13   -0.293083
2011-11-20   -0.059881
2011-11-27    1.252450
Freq: W-SUN, dtype: float64
'''


ts2['2011-11':'2011-12']
'''
2011-11-06    0.437823
2011-11-13   -0.293083
2011-11-20   -0.059881
2011-11-27    1.252450
2011-12-04    0.046611
2011-12-11    0.059478
2011-12-18   -0.286539
2011-12-25    0.841669
Freq: W-SUN, dtype: float64
'''
```

## [时序数据方法](https://gairuo.com/p/pandas-time-methods)

### rolling滑窗

```python
np.random.seed(999)
s = pd.Series(np.random.randint(0, 10, 365),
index=pd.date_range("20210101","20211231"))
s = s.sample(250, random_state=999).sort_index()

# 从当前日期往前选择一个跨度为7天的窗口
s.rolling("7D").sum().head(8)
```

### shift移动

```python
rng = pd.date_range("2026-01-01", "2026-01-03")
ts = pd.Series(range(len(rng)), index=rng)

ts.shift(3, freq=pd.offsets.BDay())
"""
2026-01-06    0
2026-01-07    1
2026-01-07    2
dtype: int64
"""

ts.shift(3, freq="BME")
"""
2026-03-31    0
2026-03-31    1
2026-03-31    2
dtype: int64
"""
```

### as_freq频率转换

> 方法内部会自动构造出一个索引作为结果序列的索引。该索引通过`pd.date_range()`构造：
>
> 以原序列的第一个值为起始值（start），以原序列的最后一个值为终止值（end），以给定的freq 为采样频率

```python
s = pd.Series(
    [1, 2, np.nan, 4],
    index=pd.to_datetime(["20210901", "20210908", "20210910", "20210915"]),
)

s.asfreq("3D", method="ffill") # 向前寻找距离该索引最近的值并以该值进行填充
'''
2021-09-01    1.0
2021-09-04    1.0
2021-09-07    1.0
2021-09-10    NaN
2021-09-13    NaN
Freq: 3D, dtype: float64
'''

s.asfreq("3D", method="bfill")
'''
2021-09-01    1.0
2021-09-04    2.0
2021-09-07    2.0
2021-09-10    NaN
2021-09-13    4.0
Freq: 3D, dtype: float64
'''
```

## [时间偏移](https://gairuo.com/p/pandas-offsets)

### Timedelta

`Timedelta`：参数weeks、days、hours、minutes、seconds、milliseconds、microseconds 和nanoseconds

```python
pd.Timedelta("1d2h")

pd.Timedelta(days=1,hours=2)


# pd.to_timedelta
# TimedeltaIndex(['1 days', '2 days', '3 days'], dtype='timedelta64[ns]', freq=None)
pd.to_timedelta([1,2,3],unit="d")


# pd.timedelta_range
pd.timedelta_range("0s","1000s",freq="6min")
pd.timedelta_range("0s","1000s",periods=3)
```

`Timedelta.components`返回类似元祖的结果

```python
pd.Timedelta("1D2h30min15s").components

# Components(days=1, hours=2, minutes=30, seconds=15, milliseconds=0, microseconds=0, nanoseconds=0)

pd.Timedelta("1D2h30min15s").components.days # 1
```

> `Timedelta`序列的dt 对象上定义的属性只有days、seconds、microseconds和nanoseconds
>
> - `seconds`：seconds 不是本身的秒数，而是对天数取余后剩余的秒数。具体地说，假设时间差为1 分6 秒，seconds 属性返回的是66
>   秒而不是6 秒
> - `total_seconds()`获取总共的秒数，对其进行60取余，可获得对应秒的数据

### DateOffset

日期偏置类型及其含义

|    偏置类型（B 表示工作日）     |         含义         |
| :-----------------------------: | :------------------: |
|             `BDay `             |        工作日        |
|             `CDay`              |       自定义日       |
|             `Week`              |       星期或周       |
|          `WeekOfMonth`          |  每月第x 周的第y 天  |
|        `LastWeekOfMonth`        | 每月最后一周的第y 天 |
|   `(B)MonthBegin/(B)MonthEnd`   |      月初/月末       |
| `(B)QuarterBegin/(B)QuarterEnd` |      季初/季末       |
|    `(B)YearBegin/(B)YearEnd`    |      年初/年末       |

`DateOffset` 类似于时间差 `Timedelta` ，但它使用日历中时间日期的规则，而不是直接进行时间性质的算术计算，让时间更符合实际生活

> `DateOffset`中例如`year`和`years`参数，`year`设置年份优先运行，`years`偏移再运行，再按时间粒度从大到小(比如先运算`years`再运算`months`)叠加偏移量

- **场景 1：跨月末的 “1 个月” 偏移（核心差异）**

```python
import pandas as pd
from datetime import datetime, timedelta

# 初始日期：1月31日（大月最后一天）
dt = datetime(2024, 1, 31)

# 1. 用DateOffset加1个月（遵循日历规则，到2月最后一天）
offset_1m = pd.DateOffset(months=1)
dt_offset = dt + offset_1m
print(f"DateOffset加1个月: {dt_offset}")  # 输出：2024-02-29 00:00:00（2024是闰年）

# 2. Timedelta无法直接加“1个月”，只能估算（比如按30天算，结果不符合日历逻辑）
dt_timedelta = dt + timedelta(days=30)
print(f"Timedelta加30天: {dt_timedelta}")  # 输出：2024-03-01 00:00:00（偏离日历的“1个月”）
```

- **场景 2：工作日偏移（DateOffset 的专属能力）**

```python
# 初始日期：2024年4月5日（周五）
dt_fri = datetime(2024, 4, 5)

# 1. DateOffset加1个工作日（跳过周六、周日，到下周一）
bday_offset = pd.offsets.BDay(1)
dt_bday = dt_fri + bday_offset
print(f"加1个工作日: {dt_bday}")  # 输出：2024-04-08 00:00:00（周一）

# 2. Timedelta加1天（只算物理时间，到周六）
dt_1day = dt_fri + timedelta(days=1)
print(f"加1天（Timedelta）: {dt_1day}")  # 输出：2024-04-06 00:00:00（周六）
```

## [时间偏移对象](https://gairuo.com/p/pandas-date-offset)

> `pd.offsets`和`pd.DateOffset`区别

- 给指定日期（2024-01-31）加 1 个月和 2 天

```python
pd.Timestamp("2024-01-31") + pd.DateOffset(months=1, days=2)

pd.Timestamp("2024-01-31") + pd.offsets.MonthEnd(1) + pd.offsets.Day(2)
```

- 给周五（2024-04-05）加 1 个工作日（应跳到下周一），给周一（2024-04-08）加 2 个工作日（应跳到周三）

```python
dt_fri = datetime(2024, 4, 5)  # 周五
dt_mon = datetime(2024, 4, 8)  # 周一

# offsets.BDay 涉及工作日优先使用offsets.BDay
dt_fri + pd.offsets.BDay(1)
dt_mon + pd.offsets.BDay(2)

# DateOffset
dt_fri + pd.DateOffset(days=1,weekday=0) # 强制跳转到周一
dt_mon + pd.DateOffset(days=2)
```

- 给 2024-01-15 跳到当月月末

```python
pd.Timestamp(2024, 1, 15) + pd.offsets.MonthEnd(1)

pd.Timestamp(2024, 1, 15) + pd.DateOffset(day=31) # day设置的参数可以设置为31,会根据实际月份天数调整
```

- 给 2024-04-05（周五）跳到当周最后一天（周日）

```python
pd.Timestamp("2024-04-05") + pd.DateOffset(days=0,weekday=6)

pd.Timestamp("2024-04-05") + pd.offsets.Week(1,weekday=6)
```

> `offsets.WeekOfMonth`：`week=0`代表是当月第一周

```python
# 3/4已超过当月第一周,故跳转到下个月第一个周一
pd.Timestamp('20260304') + pd.offsets.WeekOfMonth(week=0,weekday=0) # Timestamp('2026-04-06 00:00:00') 

pd.Timestamp('20260304') + pd.offsets.WeekOfMonth(week=1,weekday=0) # Timestamp('2026-03-09 00:00:00')
```

### 相关查询

```python
i = pd.date_range("2018-04-09", periods=4, freq="2D 2h")
ts = pd.DataFrame({"A": [1, 2, 3, 4]}, index=i)

ts.at_time("2:00")

ts.between_time("0:15", "2:45")
```

## [锚定偏移](https://gairuo.com/p/pandas-anchored-offsets)

```python
# n = 0，如果在锚点上，则日期不会移动，否则它将前滚到下一个锚点
pd.Timestamp("2014-01-02") + pd.offsets.MonthBegin(n=0) # Timestamp('2014-02-01 00:00:00')

pd.Timestamp("2014-01-02") + pd.offsets.MonthEnd(n=0) # Timestamp('2014-01-31 00:00:00')

pd.Timestamp("2014-01-01") + pd.offsets.MonthBegin(n=0) # Timestamp('2014-01-01 00:00:00')

pd.Timestamp("2014-01-31") + pd.offsets.MonthEnd(n=0) # Timestamp('2014-01-31 00:00:00')

# n != 0
pd.Timestamp("2014-01-02") + pd.offsets.MonthEnd(n=1) # 2014-01-31

pd.Timestamp("2014-01-31") + pd.offsets.MonthEnd(n=1) # 2014-02-28 在锚点会继续找下一个
```

## [时长频率单位转换](https://gairuo.com/p/pandas-frequency-conversion)

> `np.timedelta64`仅支持'W', 'D', 'h', 'm', 's', 'ms', 'us', 'ns'等固定的日期单位

```python
idx = pd.to_timedelta([1,2,3],unit="d") / np.timedelta64(1,"h") # Index([24.0, 48.0, 72.0], dtype='float64')

# 等效于下方写法
pd.to_timedelta([1, 2, 3], unit="d") / pd.Timedelta("1h")
```

## [周期的操作](https://gairuo.com/p/pandas-period-calculation)

> 理解`pd.period`

```python
p = pd.Period("2011", freq="Y-JAN") # 2011年1月作为整年的最后一月进行计算

p.start_time, p.end_time  # (Timestamp('2010-02-01 00:00:00'), Timestamp('2011-01-31 23:59:59.999999999'))

p = pd.Period("2011Q4", freq="Q-MAR") # 2011年3月作为最后一月进行计算

p.start_time, p.end_time  # (Timestamp('2011-01-01 00:00:00'), Timestamp('2011-03-31 23:59:59.999999999'))
```

```python
p = pd.Period('2012-01', freq='2M')
p + 2 # 加两个周期，到五月据的周期
# Period('2012-05', '2M')

p - 1 # 减去一个周期
# Period('2011-11', '2M')
```

## [周期类型及转换](https://gairuo.com/p/pandas-period-conversion)

```python
p = pd.Period('2011', freq='Y-DEC')

p.asfreq("M", how="start")  # Period('2011-01', 'M')
p.asfreq("M", how="end")  # Period('2011-12', 'M')	
```

## [时间重采样](https://gairuo.com/p/pandas-resampling)

[resample函数](https://gairuo.com/p/pandas-resample)

- **`closed`**:
  - 类型：`{'right', 'left'}`
  - 说明：定义时间间隔的哪一端是关闭的
- **`label`**:
  - 类型：`{'right', 'left'}`
  - 说明：在时间间隔内标记的哪一端（即时间戳将代表区间的起点还是终点）
- **`on`**:
  - 类型：字符串
  - 说明：指定时间序列的列名，适用于非索引的时间序列
- **`level`**:
  - 类型：整数或字符串
  - 说明：当索引是多层索引时，可以指定级别进行重采样
- **`origin`**:
  - 类型：`{'epoch', 'start', 'start_day','end','end_day'}` 或者 时间戳字符串
  - 说明：用于设置时间偏移的起点
    - 'epoch'：’1970-01-01‘
    - 'start'：时序序列的第一个值
    - ‘start_day'：时间序列的第一天的午夜

```python
rng = pd.date_range('1/1/2012', periods=1000, freq='S')
ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)

# 每5分钟进行一次聚合
ts.resample('5Min').sum()
'''
2012-01-01 00:00:00    74477
2012-01-01 00:05:00    74834
2012-01-01 00:10:00    76489
2012-01-01 00:15:00    25095
Freq: 5T, dtype: int64
'''

# 可以将 closed 设置为“ left”或“ right”，以指定关闭区间的哪一端
ts.resample('5Min', closed='right').mean()
ts.resample('5Min', closed='left').mean()

# 呈现开盘价、最高价、最低价、收盘价
ts.resample("5Min").ohlc()

'''                     open  high  low  close
2012-01-01 00:00:00    84   499    5    435
2012-01-01 00:05:00   229   497    1    440
2012-01-01 00:10:00   171   499    1    201
2012-01-01 00:15:00   431   499    0    289
'''
```

## [区间间隔 pd.Interval](https://gairuo.com/p/pandas-interval)

```python
# 语法
iv = pd.Interval(left=0, right=5, closed='right')

iv.length # 5

3.5 in iv # True
5.5 in iv # False

year_2020 = pd.Interval(pd.Timestamp('2020-01-01 00:00:00'),
                        pd.Timestamp('2021-01-01 00:00:00'),
                        closed='left')
```

### 间隔部分重叠

```python
i1 = pd.Interval(0, 2)
i2 = pd.Interval(1, 3)
i1.overlaps(i2) # True

i3 = pd.Interval(4, 5)
i1.overlaps(i3) # False

i4 = pd.Interval(0, 1, closed='both')
i5 = pd.Interval(1, 2, closed='both')
i4.overlaps(i5) # True
```

### 计算操作

```python
iv
# Interval(0, 5, closed='right')

shifted_iv = iv + 3
shifted_iv
# Interval(3, 8, closed='right')

extended_iv = iv * 10.0
extended_iv
# Interval(0.0, 50.0, closed='right')
```

### `pd.IntervalIndex`创建

- `from_breaks()`

- `from_arrays()`

- `from_tuples()`

- `interval_range()`

```python
pd.IntervalIndex.from_breaks([1, 3, 6, 10], closed="both")
# IntervalIndex([[1, 3], [3, 6], [6, 10]], dtype='interval[int64, both]')


pd.IntervalIndex.from_arrays(left=[1, 3, 6, 10], right=[5, 4, 9, 11], closed="neither")
# IntervalIndex([(1, 5), (3, 4), (6, 9), (10, 11)], dtype='interval[int64, neither]')


pd.IntervalIndex.from_tuples([(1, 5), (3, 4), (6, 9), (10, 11)], closed="neither")
# IntervalIndex([(1, 5), (3, 4), (6, 9), (10, 11)], dtype='interval[int64, neither]')


pd.interval_range(start=1, end=5, periods=8,closed="left")
"""
IntervalIndex([[1.0, 1.5), [1.5, 2.0), [2.0, 2.5), [2.5, 3.0), [3.0, 3.5),
               [3.5, 4.0), [4.0, 4.5), [4.5, 5.0)],
              dtype='interval[float64, left]')
"""

pd.interval_range(end=5, periods=8, freq=0.5)
```

> 方法

```python
id_interval = pd.IntervalIndex(pd.cut(s, 3))

id_demo = id_interval[:5] # 选出前5 个展示
'''
IntervalIndex([(33.945, 52.333], (52.333, 70.667], (70.667, 89.0],
(33.945, 52.333], (70.667, 89.0]],
dtype='interval[float64, right]', name='Weight')
'''


id_demo.left # Float64Index([33.945, 52.333, 70.667, 33.945, 70.667], dtype='float64')
id_demo.right
id_demo.mid
id_demo.length


id_demo.contains(4) # array([False, False, False, False, False])
id_demo.overlaps(pd.Interval(40,60)) # array([ True, True, False, True, False])
```

---

# 文本处理

## 正则表达式

```python
reg, string = "Apple", "Apple! This Is an Apple!"

re.findall(reg, string)  # ['Apple', 'Apple']
```

### 元字符

> 集合元字符：用一个元字符来匹配某个字符的集合

|  语法   |                           匹配对象                           |
| :-----: | :----------------------------------------------------------: |
|    .    |                    除换行符以外的所有字符                    |
| `\w \W` |                  字母和数字、非字母和非数字                  |
| `\d \D` |                         数字、非数字                         |
| `\s \S` | 空白字符（含空格“ ”、换行符“\n”、换页符“\f”、制表符“\t”）、非空白字符 |
| `[...]` |                方括号内“...”中的任意一个字符                 |

> 频次元字符：控制前面一个字符出现的次数
>
> **正则匹配默认使用贪婪模式，即尽可能长地匹配对应模式，如果一旦匹配到对应的正则表达式就停止匹配，可以在频次元字符后加“?”，将贪婪模式切换为懒惰模式**

|  语法   |      匹配对象      |
| :-----: | :----------------: |
|   `?`   |   出现0 次或1 次   |
|   `*`   | 出现0 次或0 次以上 |
|   `+`   | 出现1 次或1 次以上 |
|  `{n}`  |      出现n 次      |
| `{n,}`  | 出现n 次或n 次以上 |
| `{n,m}` |    出现n～m 次     |

> 逻辑元字符：主要指`|`表示的选择符号以及方括号集合匹配中`^`表示的取反符号

```python
re.findall("ab|cd", "abc bcd acd") # ["ab","cd","cd"]

re.findall("a[^\w!]", "a ab a? a!") # ["a ","a?"]
```

> 位置元字符：不匹配任何元素而是匹配位置，这里的位置包括单词的起始或终止、字符串的起始或终止
>
> **`^`和`$`针对整个字符串的开头 / 结尾（而非单词的开头 / 结尾）**

| 语法 |       匹配位置       |
| :--: | :------------------: |
| `\b` |   单词的起始或终止   |
| `\B` | 不是单词的起始或终止 |
| `^`  |     字符串的起始     |
| `$`  |     字符串的终止     |

```python
re.findall(r"\BAB|cc\b", "CAB ccd AB cc AC") # ["AB","cc"]

re.findall("^[Tt]able|ble$|Ble$", "Table Table table taBle") # ["Table","Ble"] 匹配到为开始的Table和最后的Ble
```

### 分组捕获与反向引用

```python
re.findall(r"(\w{3})-(\d{3})", "ABC-012 DEF-345")  # [('ABC', '012'), ('DEF', '345')]


# 不需要某个分组时，可以选择使用(?:...)的语法在结果中去除这个组
re.findall(r"(?:\w{3})-(\d{3})", "ABC-012 DEF-345")  # ['012', '345']


# 当一个子结构被捕获时，在它之后可以使用反斜杠加数字来引用被捕获到结果
re.findall(r"(\w:\s)(\w+)\s\1(\w+)", "A: apple A: 苹果 B: banana B: 香蕉") 
# [('A: ', 'apple', '苹果'), ('B: ', 'banana', '香蕉')]


re.findall(r"(\d)(\1)(\2{0,2})", "11222333344444") # [('1', '1', ''), ('2', '2', '2'), ('3', '3', '33'), ('4', '4', '44')]
```

> `re.sub`：正则替换

```python
re.sub(r"\d", "*", "1f93fdf0wef") # "*f**fdf*wef"

re.sub("\d", "", "1f93fdf0wef") # 'ffdfwef'

re.sub("([a-zA-Z])t", r"\1\1", "atjsjtjst")  # 'aajsjjjss'
```

### 零宽断言

|     名称     |    语法    |           功能            |
| :----------: | :--------: | :-----------------------: |
| 正向先行断言 | `(?=...)`  |  匹配...表达式的起始位置  |
| 正向后行断言 | `(?<=...)` |  匹配...表达式的终止位置  |
| 负向先行断言 | `(?!...)`  | 匹配非...表达式的起始位置 |
| 负向后行断言 | `(?<!...)` | 匹配非...表达式的终止位置 |

```python
re.findall(r"：(\d+)(?=件)", "商品成交量：758件") # ["758"]

re.findall(r"(?<=成交量：)(\d+)", "商品成交量：758 件") # ["758"]

re.findall(r"(\d+)(?![件|\d])", "交易量减少50件，交易额增加100元") # ['100']

re.findall(r"(?<![少|\d])(\d+)", "交易量减少50件，交易额增加100元") # ['100']
```

### 优先级

|   运算符(优先级由高到低)    |                             描述                             |
| :-------------------------: | :----------------------------------------------------------: |
|              \              |                            转义符                            |
|     (), (?:), (?=), []      |                        圆括号和方括号                        |
|  *, +, ?, {n}, {n,}, {n,m}  |                            限定符                            |
| ^, $, \任何元字符、任何字符 |                定位点和序列（即：位置和顺序）                |
|             \|              | 替换，"或"操作 字符具有高于替换运算符的优先级，使得"m\|food"匹配"m"或"food" |

---

> **特殊情况：使用str 对象的必要条件是序列中能够被索引或切片**

```python
s = pd.Series([{1: "temp_1", 2: "temp_2"}, ["a", "b"], 0.5, "my_string"])

s.str[1]
'''
0    temp_1
1         b
2       NaN
3         y
dtype: object
'''
```

## [文本分割](https://gairuo.com/p/pandas-split-strings)

```python
s2 = pd.Series(['a_b_c', 'c_d_e', np.nan, 'f_g_h'], dtype="string")

# 指定展开列数，n 为切片右值
s2.str.split('_', expand=True, n=1)

'''
      0     1
0     a   b_c
1     c   d_e
2  <NA>  <NA>
3     f   g_h
'''
```

> 使用正则

```python
s = pd.Series(["1+1=2"])
s.str.split(r"\+|=", expand=True) # 使用+或者=作为分隔符
```

> 划分：将文本按分隔符号划分为三个部分，保留分隔符

```python
s = pd.Series(['Linda van der Berg', 'George Pitt-Rivers'])

s.str.partition("-")

'''
	0	1	2
0	Linda van der Berg		
1	George Pitt	-	Rivers
'''
```

## [文本替换](https://gairuo.com/p/pandas-replace-strings)

```python
s = pd.Series(["12", "-$10", "$10,000"], dtype="string")

s.str.replace("$","")
```

> 使用正则
>
> `group`参数理解
>
> - `.group(0)`（或简写为`m.group()`）：代表**整个正则表达式匹配到的完整字符串**（这是最常用的场景）
>
> - 如果正则里有分组（用`()`括起来的部分），`group(1)`、`group(2)`会对应第 1、第 2 个分组的内容（本例没有分组，所以只用`group(0)`）
> - 每次匹配到内容，就会调用匿名函数，返回对应内容

```python
(
  pd.Series(["foo", "fuz", np.nan])
  .str.replace(r"[a-z]+", lambda m: m.group(0)[::-1],regex=True)
)
'''
0    oof
1    zuf
2    NaN
dtype: object
'''

# 取替换成第二部分并大小写互换
pat = r"(?P<one>\w+) (?P<two>\w+) (?P<three>\w+)"
repl = lambda m: m.group("two").swapcase()
pd.Series(["One Two Three", "Foo Bar Baz"]).str.replace(pat, repl,regex=True)
"""
0    tWO
1    bAR
dtype: object
"""
```

> 指定替换

```python
s = pd.Series(['a', 'ab', 'abc', 'abdc', 'abcde'])
# 保留第一个，其他的替换或者追加 X
s.str.slice_replace(1, repl='X')
'''
0    aX
1    aX
2    aX
3    aX
4    aX
dtype: object
'''
# 指定位置前删除并用 X 替换
s.str.slice_replace(stop=2, repl='X')
'''
0       X
1       X
2      Xc
3     Xdc
4    Xcde
dtype: object
'''
# 指定区间的内容被替换
s.str.slice_replace(start=1, stop=3, repl='X')
'''
0      aX
1      aX
2      aX
3     aXc
4    aXde
dtype: object
'''
```

> 重复替换

```python
# 对整体重复两次
pd.Series(['a', 'b', 'c']).repeat(repeats=2)

# 对每个行内的内容重复两次
pd.Series(['a', 'b', 'c']).str.repeat(repeats=2)

# 指定每行重复几次
pd.Series(['a', 'b', 'c']).str.repeat(repeats=[1, 2, 3])
'''
0      a
1     bb
2    ccc
dtype: object
'''
```

## [文本连接](https://gairuo.com/p/pandas-concat-strings)

### str.cat

```python
s = pd.Series(['a', 'b', 'c', 'd'], dtype="string")
s.str.cat(sep=',') # 'a,b,c,d'
s.str.cat() # 'abcd'

# 对空值处理
t = pd.Series(['a', 'b', np.nan, 'd'], dtype="string")
t.str.cat(sep=',') #'a,b,d'
t.str.cat(sep=',', na_rep='-') # 'a,b,-,d'

# 指定列表序列连接
s.str.cat(['A', 'B', 'C', 'D'])
'''
0    aA
1    bB
2    cC
3    dD
dtype: string
'''
```

### str.join

```python
s = pd.Series(["aaa","bb","c"])

s.str.join("-")
'''
0    a-a-a
1      b-b
2        c
dtype: object
'''
```

## [文本查询匹配](https://gairuo.com/p/pandas-match-string)

### str.extract

```python
(pd.Series(['a1', 'b2', 'c3'],
          dtype="string")
 .str
 .extract(r'([ab])(\d)', expand=True)
)
'''
      0     1
0     a     1
1     b     2
2  <NA>  <NA>
'''

# 取分组名称
s.str.extract(r'(?P<letter>[ab])(?P<digit>\d)')
'''
  letter digit
0      a     1
1      b     2
2    NaN   NaN
'''
```

### str.extractall

```python
s = pd.Series(["a1a2", "b1b7", "c1"],
              index=["A", "B", "C"],
              dtype="string")
two_groups = '(?P<letter>[a-z])(?P<digit>[0-9])'

s.str.extractall(two_groups)
'''
        letter digit
  match
A 0          a     1
  1          a     2
B 0          b     1
  1          b     7
C 0          c     1
'''
```

## [文本常用方法](https://gairuo.com/p/pandas-strings-attributes)

```python
# 指定字母的数量
s.str.count('a')

# 支持正则，包含 abc 三个字母的总数
s.str.count(r'a|b|c')

# 字符长度
s.str.len()
```

```python
# 检查字母和数字字符
s.str.isalpha() # 是否纯英文字母组成
s.str.isalnum() # 是否单词、数字或者它们组合形式组成
# 请注意，对于字母数字检查，针对混合了任何额外标点或空格的字符的检查将计算为 False

s.str.isdecimal() # 是否数字 0-9 组成合规10进制数字
s.str.isdigit() # 同 但可识别 unicode中的上标和下标数字
s.str.isnumeric() # 是否可识别为一个数字，同 isdigit 可识别分数
```

---

# 样式

## [内置样式](https://gairuo.com/p/pandas-building-styles)

```python
# 空值高亮 highlight_null
df.style.highlight_null(color="red",subset=["team"]) # 默认为背景红色
df.style.highlight_null(props="font-weight:700;color:red") # 引入css

# 最值高亮 highlight_max/min
# 使用 pd.IndexSlice 索引器（和 loc[] 类似）
# 注意，数据是所有数据，算最值的范围不是全部
(
  df.head(10).style
  .highlight_max(subset=pd.IndexSlice[:10,"Q1":"Q4"], color="green", axis=1)
  .highlight_min(subset=pd.IndexSlice[:10,"Q1":"Q4"],axis=1,props="background-color:yellow;")
)

# 区间高亮 highlight_between
df.style.highlight_between(subset=pd.IndexSlice[:,"Q1":"Q4"],left=10,right=90,color="yellow",axis=1,inclusive="both")

# 分位数高亮 highlight_quantile
df = pd.DataFrame(np.arange(10).reshape(2,5) + 1)
# 将后四分位的数据高亮，即后 3/4 的数据
df.style.highlight_quantile(axis=None, q_left=0.25, color="#fffd75")
# 按行或按列高亮显示分位数，在本例中按行高亮显示
df.style.highlight_quantile(axis=1, q_left=0.8, color="#fffd75")
```

[色彩映射参考-cmap](https://matplotlib.org/devdocs/gallery/color/colormap_reference.html)

```python
# 文本颜色渐变 text_gradient
# 数据集
df = pd.DataFrame(columns=["City", "Temp (c)", "Rain (mm)", "Wind (m/s)"],
                  data=[["Stockholm", 21.6, 5.0, 3.2],
                        ["Oslo", 22.4, 13.3, 3.1],
                        ["Copenhagen", 24.5, 0.0, 6.7]])
# 按列着色显示值，axis=0，预选数值列
df.style.text_gradient(axis=0)
# 使用 axis=None 对所有值进行整体着色
df.style.text_gradient(axis=None)
# 从低值和高值颜色着色
df.style.text_gradient(axis=None, low=0.75, high=1.0)
# 手动设置vmin和vmax梯度阈值
df.style.text_gradient(axis=None, vmin=6.7, vmax=21.6)


# 背景渐变 background_gradient
(
    df.head(15)
    .style.background_gradient(subset=["Q1"], cmap="spring")  # 指定色系
    .background_gradient(subset=["Q2"], vmin=60, vmax=100)  # 指定应用值区间
    .background_gradient(subset=["Q3"], low=0.6, high=0)  # 高低百分比范围
    .background_gradient(subset=["Q4"], text_color_threshold=0.9)  # 文本色深,方便显示数据
)

# 条形图 bar
# 指定应用范围
df.style.bar(subset=['Q1'])
# 定义颜色
df.style.bar(color='green')
df.style.bar(color='#ff11bb')
# 以向进行计算展示
df.style.bar(axis=1)
# 样式在格中的占位百分比，0-100, 100占满
df.style.bar(width=80) # height表示条形图的粗细
# 对齐方式：
# ‘left’ 最小值开始,
# ‘zero’ 0值在中间,
# ’mid’  (max-min)/2 值在中间，负（正）值0在右（左）
df.style.bar(align='mid')
# 大小基准值
df.style.bar(vmin=60, vmax=100)
```

## [显示格式](https://gairuo.com/p/pandas-style-format)

[Python格式化字符串](https://gairuo.com/p/python-format-string)

```python
# 宽度：字符串默认在后面补位，数字在前面补位
name_1 = "tom"
name_2 = "lily"

print(f"{name_1:5}100分")
print(f"{name_2:5}100分")

print(f"{name_1:>08}")  # 00000tom >在前边补0
print(f"{name_1:<08}")  # tom00000 < 在后面补0
print(f"{name_1:^08}")  # 00tom000 ^ 居中两端补0 

f'今年{age:08}岁' # 加0不够长度前边补0

# 数字格式化
print(f'最高{8848:+}m') # 显示正号
print(f'最低{-11043: }m') # 空格, 正数前导空格，负数使用减号
# 最高+8848m
# 最低-11043m

a = 123.456
f'a is {a:.2f}' # 两位小数
# a is 123.46

f'a is {a:8.2f}' # 8个字符位置, 不够用空格占位
# a is   123.46

f'a is {a:08.2f}' # 8个字符位置, 不够用0占位
# a is 00123.46

f'a is {a:8.2%}' # 共8位, 不足后边补0, 再加百分号
# a is 12345.60%

f'a is {3253547568.43:,f}' # 加千分位
# a is 3,253,547,568.430000
```

### style.format

```python
# 百分号，类似 29.57%
df.style.format("{:.2%}")

# 指定列全变为大写
df.style.format({'name': str.upper})

# B 保留四位，D 两位小数并显示正负号
df.style.format({'B': "{:0<4.0f}", 'D': '{:+.2f}'})

# 应用 lambda
df.style.format({"B": lambda x: "±{:.2f}".format(abs(x))})

# 缺失值的显示格式
df.style.format("{:.2%}", na_rep="-")

# 替换空值，精度保留3位小数（1.3.0+）
df.style.format(na_rep='MISS', precision=3)

# 转义方法，还可以用 LaTeX（1.3.0+）
df.style.format('<a href="a.com/{0}">{0}</a>', escape="html")

# 对float数据的整数个小数部分的分隔，默认是小数点（1.3.0+）
df.loc[:,'Q2':].astype(float).style.format(decimal=';')

# 处理内置样式函数的缺失值
df.style.highlight_max().format(None, na_rep="-")
```

> **注意事项：新版本的pandas中不能链式调用format会导致覆盖(而不是合并)，需要将所有的格式化放置于一个字典中**

```python
# 链式方法使用格式
(df.head(15)
 .head(10)
 .assign(avg=df.mean(axis=1, numeric_only=True)/100) # 增加平均值百分比
 .assign(diff=lambda x: x.avg.diff()) # 和前位同学差值
 .style
 .format({'name': str.upper,'avg': "{:.2%}",'diff': "¥{:.2f}"}, na_rep="-")
)
```

## [样式配置操作](https://gairuo.com/p/pandas-style-options)

```python
# 给表格一个标题
df.style.set_caption('学生成绩表')

# 表格样式

# 设置内联样式 set_properties 使用css样式
# 指定列，设置字色为红色
df.style.set_properties(subset=['Q1'], **{'color': 'red'})
# 一些其他示例
df.style.set_properties(color="white", align="right")
df.style.set_properties(**{'background-color': 'yellow'})
df.style.set_properties(**{'width': '100px', 'font-size': '18px'})
df.style.set_properties(**{'background-color': 'black',
                           'color': 'lawngreen',
                           'border-color': 'white'})

# 表格属性 set_table_attributes：给 <table> 标签增加属性，可以随意给定属性名和属性值
df.style.set_table_attributes('class="pure-table"')
df.style.set_table_attributes('id="gairuo-table"')

# 表格样式属性指定 set_table_styles
df.style.set_table_styles(
    [{"selector": "tr:hover", "props": [("background-color", "yellow")]}] # 鼠标移动上去整行背景变黄
)

df.style.set_table_styles({
    'A': [{'selector': '',
           'props': [('color', 'red')]}],
    'B': [{'selector': 'td',
           'props': [('color', 'blue')]}]
}, overwrite=False)

df.style.set_table_styles({
    0: [{'selector': 'td:hover',
         'props': [('font-size', '25px')]}]
}, axis=1, overwrite=False)
```

## [样式应用函数](https://gairuo.com/p/pandas-style-function)

```python
# 最大值显示红色
def highlight_max(x):
    return ['color: red' if v == x.max() else '' for v in x]

# 应用函数apply,输出为对应序列长度的格式
df.style.apply(highlight_max)
# 按行应用
df.loc[:,'Q1':'Q4'].style.apply(highlight_max, axis=1)

# 对整行进行格式设置
yellow_css = 'background-color: yellow'
sfun = lambda x: [yellow_css]*len(x) if x.math > 80 else ['']*len(x)
(df.style
 .apply(sfun, axis=1)
)

# 应用函数map
yellow_css = "background-color: yellow"
sfun = lambda x: yellow_css if type(x) == int and x > 90 else ""
(df.style.map(sfun))
```

> subset 可传入索引器 `pd.IndexSlice`实现复杂逻辑下的部分区域应用

```python
# 筛选条件1：满足指定条件并且应用在某个列上
sst1 = pd.IndexSlice[df.loc[(df.c=='good') & (df.f>60)].index, 'f']
# 样式方法
sfun1 = lambda x: 'background-color: yellow' if x > 500 else ''

(
    df.style
    .map(sfun1, subset=sst1)
#   .to_excel('gairuo.xlsx', engine='openpyxl')
)
```

> 样式复用/清除

```python
# 将 df 的样式赋值给变量
style1 = df.style.map(color_negative_red)
# df2 的样式为 style2
style2 = df2.style
# style2 使用 style1的样式
style2.use(style1.export())


dfs = df.loc[:,'Q1':'Q4'].style.apply(highlight_max)
dfs.clear() # 清除
dfs # 此时 dfs 不带任何样式，但还是 Styler 对象
```

---