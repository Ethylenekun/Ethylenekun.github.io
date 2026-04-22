---
title: Pandas习题(时间日期、文本处理、样式、可视化、分类)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas习题，在日常过程中遇见的比较不错的习题
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 6
abbrlink: fea6abb8
---
# Pandas习题 - 时间日期、文本处理、样式、可视化、分类

## [日志数据清洗与分析](https://gairuo.com/m/pandas-log-cleaning-extract)

> 题目：
>
> 1. 统计每个用户访问页面的次数
> 2. 找出访问量最高的页面
> 3. 将时间字段转换为 pandas 的 `datetime` 类型，以便后续做时间分析

```python
from io import StringIO

log_data = StringIO("""
[2024-11-05 09:15:32] user: alice visited /home
[2024-11-05 09:18:01] user: bob visited /products
[2024-11-05 09:19:55] user: alice visited /products
[2024-11-05 09:20:10] user: charlie visited /home
[2024-11-05 09:25:42] user: bob visited /checkout
[2024-11-05 09:28:15] user: alice visited /checkout
[2024-11-05 09:35:00] user: charlie visited /products
""")

df = pd.read_csv(log_data, header=None, names=["raw"])
```

> 解法

```python
# 提取时间、用户和页面
parsed = (
    df['raw']
    .str.extract(r'\[(?P<timestamp>.*?)\]\s+user:\s+(?P<user>\w+)\s+visited\s+(?P<page>\S+)')
)

# 转换时间类型
parsed['timestamp'] = pd.to_datetime(parsed['timestamp'])

# 统计每个用户访问次数
user_visits = parsed['user'].value_counts().reset_index()
user_visits.columns = ['user', 'visit_count']

# 统计每个页面访问次数
page_visits = parsed['page'].value_counts().reset_index()
page_visits.columns = ['page', 'visit_count']
```

---

## [替换手机号中间四位](https://gairuo.com/m/pandas-quiz-225-e300)

> 题目：用`str.replace()`方法将手机号中间4位替换为`****`

```python
data = {'phone': ['13800138000',
                  '13912345678',
                  '15098765432',
                  '18600001111']}

df = pd.DataFrame(data)
```

> 解法

```python
df['phone'].str.replace(r"(\d{3})(\d+)(\d{4})",r"\1****\2",regex=True)

df['phone'].str.replace(r"(\d{3})(\d+)(\d{4})",lambda x:f"{x.group(1)}****{x.group(2)}",regex=True)
```

---

## [将类似字典的文本转为列](https://gairuo.com/m/pandas-turns-dictionary-like-text-columns)

> 题目

```python
from io import StringIO

data= """
col
A:10,B:20,C:30
A:20,D:50,E:60
"""

pd.read_csv(StringIO(data),sep=r"\s+")

# 所需结果
'''
A B C D E
10 20 30 null null
20 null null 50 60
'''
```

> 解法

```python
# 个人解法
(
    df["col"]
    .str.split(",")
    .explode()
    .str.extract(r"(?P<letter>[A-Z]):(?P<number>\d+)")
    .reset_index()
    .pivot(index="index", columns="letter", values="number")
    .rename_axis(None, axis=0)
    .rename_axis(None, axis=1)
)

# 官方解法
(
    df.col.str.replace(':', '=')
    .map(lambda x: f'dict({x})')
    .map(eval)
    .apply(pd.Series)
)
```

---

## [将月份转为两位数字](https://gairuo.com/m/pandas-turns-month-into-two-digits)

> 题目：月份信息转为两位数字，即 1 至 9 月的数字前补 0

```python
from io import StringIO

data = """
时间
2020年12月上半月
2020年12月下半月
2020年8月上半月
2020年8月下半月
2020年9月上半月
2020年9月下半月
2021年10月上半月
2021年12月下半月
"""

df = pd.read_csv(StringIO(data),sep=r"\s+")
```

> 解法

```python
pat = r"(?P<a>\w+年)(?P<b>\w+)(?P<c>月\w+)"
repl = lambda m: m.group('a')+m.group('b').zfill(2)+m.group('c')

df.时间.str.replace(pat, repl, regex=True)

# 正则断言
df["时间"].str.replace(r"(?<=年)(\d+)(?=月)",lambda x:x.group(1).zfill(2),regex=True)
```

---

## [保留所有无子级分类](https://gairuo.com/m/pandas-filtering-non-child-classification)

> 题目：需要保留所有末端分叉，主干和枝干全部删除

```python
from io import StringIO

data = '''
level
1
1.1
2
2.1
2.2
3
3.1
3.1.1
3.1.1.1
3.1.1.2
3.1.1.10
3.1.1.101
3.2.1
'''

df = pd.read_csv(StringIO(data), sep='\t')
```

> 解法

```python
# 将点之间的数据哈希 hash，就会消除字面特性
df.level.map(lambda x: '.'.join([str(hash(i)) for i in x.split('.')]))

# 以上代码相当于：
def hashed(val: str):
    val = val.split('.') # 拆为列表
    val = map(hash, val) # 哈希
    val = map(str, val) # 转为字符
    return '.'.join(val)

ser = df.level.map(hashed)

# 算法：如果它没有子层，则以它为开头的只有自己一个
df[ser.map(lambda x: ser.str.startswith(x).sum() == 1)]
```

---

## [将天数转换为指定格式的小时数](https://gairuo.com/m/pandas-days-hours-format)

> 题目：增加一个 new 列，将 time 列的时长显示为 `小时:分钟:秒` 这样的格式

```python
data = {'id': [1123, 2342],
        'time': ['2 days 03:29:05', '1 days 01:57:53']
       }
df = pd.DataFrame(data)
df['time'] = pd.to_timedelta(df['time'])
```

> 解法
>
> - `.components` 是一个属性，它返回一个 `DataFrame`，其中包含 `timedelta` 对象的各个组件：天数、小时、分钟和秒

```python
# 个人解法
def f(ts:int):
    hour = int(ts / 60 // 60)
    min = int((ts - hour * 60 * 60) // 60)
    second = int((ts - hour * 60 * 60 - min * 60))
    return f"{hour:02}:{min:02}:{second:02}"

df['time'].dt.total_seconds().map(int).map(f)

# 官方解法
new = (
    df.time.dt
    .components
    .apply(lambda x: f'{x.days*24+x.hours:02}:{x.minutes:02}:{x.seconds:02}',
           axis=1)
)
df.assign(new=new)
```

---

## [客服对话文本分析](https://gairuo.com/m/pandas-customer-service-dialogue-text)

> 题目：聊天内容前有一个 tab 符，首尾两句是系统自动提示，用“ >> ”标识，客服名称开头有“客服”字样，现在需要知道首次响应时长和平均响应时长

```python
data = """
对话开始 >>
李庆辉 2020-05-15 12:33:50
	你好，可以退货吗
客服999 2020-05-15 12:33:55 >>
	工号999很高兴为您服务~。
客服999 2020-05-15 12:33:53
	您好
客服999 2020-05-15 12:34:04
	您可以自己操作申请取消订单的。
李庆辉 2020-05-15 12:34:04
	退款多久到账呢？
客服999 2020-05-15 12:34:28
	一般1-7个工作日
李庆辉 2020-05-15 12:35:01
	OMG! 好久呢
李庆辉 2020-05-15 12:40:55
	能不能快点
客服999 2020-05-15 12:42:23
	一般情况下很快就会到账的。
李庆辉 2020-05-15 12:43:04
	OMG! 好久呢
客服999 2020-05-15 12:44:01
	一般情况下很快就会到账的。
对话结束  >>
	长时间未回复，对话结束
"""

from io import StringIO
df = pd.read_csv(StringIO(data),names=["chats"],dtype="string")
```

> 解法

```python
(
    df.loc[~(df["chats"].str.startswith("\t")) & ~(df["chats"].str.endswith(">>"))][
        "chats"
    ].str.extract(r"(?P<name>\w+)\s(?P<dt>.*)")
    .assign(dt = lambda x:x["dt"].pipe(pd.to_datetime))
    .loc[lambda x:x['name'].ne(x['name'].shift(fill_value=""))]
    .assign(diff = lambda x:x["dt"].diff())
    .assign(avg = lambda x:x["diff"].mean().total_seconds())
)
```

---

## [按每两分钟采样形成稀疏矩阵](https://gairuo.com/m/pandas-samples-every-minutes-sparse-matrix)

> 题目

```python
from io import StringIO

data = """
date	time	content
2021/8/12	22:22:34.722+8:00	A
2021/8/12	22:23:34.222+8:00	B
2021/8/12	22:34:45.213+8:00	A
2021/8/12	22:35:34.722+8:00	D
2021/8/12	22:36:34.722+8:00	C
2021/8/12	22:37:34.722+8:00	E
2021/8/12	22:38:34.722+8:00	E
2021/8/12	22:39:34.722+8:00	F
2021/8/12	22:40:34.423+8:00	E
2021/8/12	22:41:34.722+8:00	C
2021/8/12	22:42:34.422+8:00	B
2021/8/12	22:43:34.722+8:00	B
2021/8/12	22:44:34.722+8:00	A
"""

df = pd.read_csv(StringIO(data),sep=r"\s+")

# 所需结果
'''
                           A  B  C  D  E  F
time
2021-08-12 22:22:00+08:00  1  1  0  0  0  0
2021-08-12 22:34:00+08:00  1  0  0  1  0  0
2021-08-12 22:36:00+08:00  0  0  1  0  1  0
2021-08-12 22:38:00+08:00  0  0  0  0  1  1
2021-08-12 22:40:00+08:00  0  0  1  0  1  0
2021-08-12 22:42:00+08:00  0  2  0  0  0  0
2021-08-12 22:44:00+08:00  1  0  0  0  0  0
'''
```

> 解法

```python
(
    df.loc[:, ["content"]]
    .assign(time=pd.to_datetime(df["date"] + " " + df["time"]))
    .groupby(pd.Grouper(freq="2min", key="time"))
    ['content'].apply(lambda x:x.pipe(pd.get_dummies))
    .groupby(level=0)
    .sum()
    .astype(int)
)
```

---

## [完成自动化交替排班](https://gairuo.com/m/python-automatic-alternate-scheduling)

> 习题：每次有四人上班，每次后边的两位接着再上一次

```python
lst = [*"abcdefgh"]
```

> 解法

```python
days = pd.Timestamp("now").daysinmonth

def f(days:int):
    
    data = []
    for _ in range(days):
        data.append(lst[:4])
        lst = lst[2:] + lst[:2] 
    return data

idx = pd.date_range(pd.Timestamp("now").replace(day=1).date(),periods=days)

pd.DataFrame(f(days),index=idx)

# pd.Series(f(days),index=idx).apply(lambda x:pd.Series(x)) 相同效果
```

---

## [计算两个日期间的季度数](https://gairuo.com/m/pandas-quarters-between-dates)

> 题目

```python
data = {
    "start": ["2022-12-01", "2023-01-01", "2023-01-01"],
    "end": ["2023-01-01", "2024-05-01", "2023-07-01"],
}

df = pd.DataFrame(data, dtype="datetime64[ns]")
```

> 解法

```python
(
    df.apply(lambda s:s.dt.to_period("Q"))
    .pipe(lambda d: d.end - d.start)
    .apply(lambda x: x.n) # n属性
)
```

---

## [指定时间为周期按天分组](https://gairuo.com/m/pandas-specifies-time-period-grouped-days)

> 题目：根据 date 列时间，计算其是否在前一天 08：00（包含）至当天 08：00（不包含）范围内 data 列值的和

```python
import io

data = """
date    data
2022-02-01 07:00    1
2022-02-01 08:00    1
2022-02-01 09:00    2
2022-02-01 14:00    1
2022-02-01 23:00    3
2022-02-02 09:00    1
2022-02-03 10:00    2
2022-02-04 11:50    3
"""
df = pd.read_csv(io.StringIO(data), sep=r"\s{2,}", engine="python")
```

> 解法：对 `.resample()` 的参数解释一下，`D` 是按天为周期，offset 为时间偏移量，默认从零点开始，由于需求是按 8:00 开始，offset 设置为 8h，包括 8 点则设 closed 为 left。on 参数为采样的时间列名

```python
(
    df.astype({'date': 'datetime64[ns]'})
    .resample('D', on='date', offset='8h', closed='left')
    .sum()
    .rename(index=lambda x: x + pd.DateOffset(days=1))
)
```

---

## [识别时间格式进行转换](https://gairuo.com/m/pandas-recognizes-time-format-conversion)

> 题目

```python
df = pd.DataFrame({'转换前': ['测试1', '测试2', *['2020/6/30']*6]})

# 所需结果
'''
         转换前     测试后
0        测试1     测试1
1        测试2     测试2
2  2020/6/30  06月30日
3  2020/6/30  06月30日
4  2020/6/30  06月30日
5  2020/6/30  06月30日
6  2020/6/30  06月30日
7  2020/6/30  06月30日
'''
```

> 解法

```python
(
  pd.to_datetime(df["转换前"],errors="coerce")
  .map(lambda x:x.strftime("%m月%d日"),na_action="ignore")
  .fillna(df["转换前"])
  .rename("测试后")
  .pipe(df.join)
)
```

---

## [数据样式综合应用](https://gairuo.com/m/pandas-quiz-235-503)

> 题目：
>
> 1. 销售额和成本列：显示为千分位分隔的货币格式（如¥1,250,000）
> 2. 利润率列：显示为百分比格式，保留1位小数
> 3. 同比增长列：正增长显示绿色，负增长显示红色
> 4. 为销售额列添加渐变色背景（数值越大颜色越深）
> 5. 设置表头样式：深色背景、白色文字、加粗

```python
data = {
    '月份': ['1月', '2月', '3月', '4月', '5月', '6月'],
    '销售额': [1250000, 980000, 1560000, 1420000, 1680000, 1950000],
    '成本': [850000, 720000, 1050000, 950000, 1150000, 1250000],
    '利润率': [0.32, 0.265, 0.327, 0.331, 0.315, 0.359],
    '同比增长': [0.08, -0.12, 0.25, 0.18, 0.22, 0.31]
}
df = pd.DataFrame(data)
df
```

> 解法

```python
(df.style
 .format({
     '销售额': '¥{:,.0f}',
     '成本': '¥{:,.0f}',
     '利润率': '{:.1%}',
     '同比增长': '{:.2%}'
 })
 .map(lambda x: 'color: green' if isinstance(x, (int, float)) and x > 0 else 
          'color: red' if isinstance(x, (int, float)) and x < 0 else '', 
          subset=['同比增长'])
 .background_gradient(subset=['销售额'], cmap='Blues')
 .set_table_styles([{
     'selector': 'th',
     'props': [('background-color', '#2c3e50'), 
              ('color', 'white'), 
              ('font-weight', 'bold')]
 }])
)
```

---

## [全国城市房价分析及样式可视化](https://gairuo.com/m/pandas-urban-house-price-style-visualization)

> 题目

```python
from io import StringIO

data = """
序号	城市名称	平均单价（元/㎡）	环比	同比
1	深圳	78,722	+2.61%	+20.44%
2	北京	63,554	-0.82%	-1.2%
3	上海	58,831	+0.4%	+9.7%
4	厦门	48,169	-0.61%	+9.52%
5	广州	38,351	-1.64%	+13.79%
6	三亚	35,981	-0.19%	-3.88%
7	南京	33,301	+1.59%	+8.02%
8	杭州	32,181	+3.11%	+4.61%
9	天津	26,397	+2.34%	+3.5%
10	福州	25,665	-1.05%	-4.1%
11	宁波	24,306	-1.43%	+13.13%
12	珠海	23,293	-1.42%	-0.49%
13	温州	23,009	+3.01%	+7.01%
14	苏州	22,540	-1.35%	-2.8%
15	青岛	21,490	-1.7%	+1.95%
16	东莞	21,391	+6%	+34.44%
17	丽水	19,775	+2.3%	-1.78%
18	武汉	19,021	-0.18%	+4.51%
19	成都	17,443	-2.41%	+11.84%
20	无锡	17,131	-0.27%	+12.5%
"""

df = pd.read_csv(StringIO(data),sep=r"\s+")
```

> 解法

```python
df = (
    # 去掉千分位符并转为整型
    df.assign(平均单价=dfr['平均单价（元/㎡）'].str.replace(',','').astype(int))
    .assign(同比=dfr.同比.str[:-1].astype(float)) # 去百分号并转为浮点型
    .assign(环比=dfr.环比.str[:-1].astype(float)) # 去百分号并转为浮点型
    .loc[:,['城市名称','平均单价','同比','环比']] # 重命名列
)

# 各城市平均房价同比与环比情况
(
    df.set_index('城市名称')
    .loc[:, '同比':'环比']
    .plot
    .bar()
)

(
    df.style
    .highlight_max(color='red', subset=['同比', '环比'])
    .highlight_min(subset=['同比', '环比'])
    .format({'平均单价':"{:,.0f}"})
    .format({'同比':"{:2}%", '环比':"{:2}%"})
)

(
    df.style
    .bar(subset=['平均单价'], color='yellow')
)

(
    df.style
    .background_gradient(subset=['平均单价'], cmap='BuGn')
    .format({'同比':"{:2}%", '环比':"{:2}%"})
    .bar(subset=['同比'], 
        color=['#ffe4e4','#bbf9ce'], # 上涨、下降的颜色
        align='zero'
        )
    .bar(subset=['环比'], 
        color=['red','green'], # 上涨、下降的颜色
        align='zero'
        )
)
```

---

## [产品类别进行标准化处理](https://gairuo.com/m/pandas-quiz-198-f6ab)

> 题目：
>
> 1. 将 category 列转换为 Categorical 类型，并指定类别顺序为：['Electronics', 'Clothing', 'Food']
> 2. 按自定义类别顺序对数据进行排序
> 3. 添加一个新列 category_code，显示每个类别对应的数字编码

```python
data = {
    'product': ['iPhone', 'T-Shirt', 'Coffee', 'Laptop', 'Jeans', 'Milk'],
    'category': ['Electronics', 'Clothing', 'Food', 'Electronics', 'Clothing', 'Food']
}
df = pd.DataFrame(data)
df
```

> 解法

```python
(
    df.assign(category=pd.Categorical(df["category"],categories=["Electronics", "Clothing", "Food"],ordered=True))
    .sort_values("category")
    .assign(category_code=lambda x: x['category'].cat.codes)
    .reset_index(drop=True)
)
```

---

## [根据时间段转换为各小时的秒数](https://gairuo.com/m/pandas-converts-period-per-hour-seconds)

> 题目：将24个小时段中每个小时所占用的秒数计算出来

```python
from io import StringIO

data = """
start_time,end_time
2/2/2021 07:24:15,2/2/2021 08:54:41
2/3/2021 11:00:05,2/3/2021 15:10:37
2/1/2021 10:40:34,2/1/2021 14:43:50
2/4/2021 10:37:37,2/4/2021 11:19:17
2/3/2021 20:37:51,2/4/2021 02:37:52
2/3/2021 21:37:52,2/3/2021 22:08:39
"""

df = pd.read_csv(StringIO(data),sep=r",",parse_dates=[0,1])
```

> 解法

```python
def duration_by_hour(start, end):
    p = pd.period_range(pd.Period(start, freq='s'), pd.Period(end, freq='s'))
    pr = {h:pd.to_timedelta(t[-1]-t[0]).seconds for h,t in p.groupby(p.hour).items()}
    return pr

s = df.apply(lambda r: duration_by_hour(r.start_time, r.end_time), axis=1)

final = {}
for d in s.to_list():
    for k in d.keys():
        final[k] = final.get(k,0) + d[k]
        
(pd.Series(final).sort_index()/60).plot.bar()

# 简化方法
def func(ser):
    s = pd.date_range(ser.start_time, ser.end_time, freq="s")
    return pd.Series(s).groupby(s.hour).count() - 1

df.apply(func, axis=1).sum().astype(int)
```

---

## [数据样式可视化案例](https://gairuo.com/m/pandas-style-demand)

> 题目：
>
> - 对最后一列，且在第 c 列对应有 demand 文本的数值，如果大于500设置黄色底纹填充
> - 对最后一列，且在第c列对应有demand文本的数值，如果大于500在后一列的单元格输入文字：警告，红色底纹填充
> - c 列往后所有列数据小于 0 的用红色字体
> - 前三列带有 A 字符的用绿色加粗字体

```python
data = '''
grade 商品 11月1日 11月2日 11月3日
NA AB demand 1136 2556 2841
Z BB 海运 2272 1000 120
ZA CA 空运 120 496 661
N BC demand -78 520 1122
Z BB demand 2369 -99 374
N ND 海运 501 498 1098
'''
```

> 解法

```python
df = pd.read_csv(StringIO(data), sep=r"\s+")
df = df.reset_index().set_axis([*"abcdef"],axis=1)

(
  df.assign(g = df.apply(lambda x:"警告" if x['c']=="demand" and x["f"]>500 else np.nan,axis=1))
  .style
  .map(lambda x:"background-color:red" if x== "警告" else "",subset="g")
  .map(lambda x:"background-color:yellow",
       subset=pd.IndexSlice[df.loc[(df.c=='demand') & (df.f>500)].index, 'f'])
  .map(lambda x:"color:red" if x<0 else "",subset=[*"def"])
  .map(lambda x:"color:green;font-weight:700" if "A" in str(x).upper() else "",subset=[*"abc"])
)
```

---

## [对有缺失月份数据求与上月温差](https://gairuo.com/m/pandas-temperature-difference-months)

> 题目：按照省份，在当月中计算出与上月的温差（即当月减去上月），可问题是在省份内，月份是不全的，如果当月没有上月数据变显示为 Nan

```python
data_dict = {"province":['河南','河北','河南','河南','河南','河北'],
             "month":['2月','1月','1月','5月','3月','2月'],
             "temperature":['43','23','34','32','23','45'] 
            }

df = pd.DataFrame(data_dict)
df
```

> 解法：补足1-12月数据后进行温差计算，再剔除不存在的月份

```python
# 个人解法
def f(i:pd.DataFrame):
  return (
    i.set_index("month")
    .reindex([f"{x}月" for x in range(1,13)])
    .assign(diff = lambda x:x["temperature"].diff())
    .dropna(subset=["temperature"])
    )

(
  df.astype({"temperature":int})
  .groupby("province")
  .apply(f).reindex(pd.MultiIndex.from_arrays([df["province"],df["month"]]))
  .reset_index()
)

# 官方解法
def f(item:pd.DataFrame):
    new = pd.DataFrame({"month":[f"{i}月" for i in range(1,13)],"temperature":np.nan})
    return (
        pd.concat([new.loc[~new["month"].isin(item["month"])], item])
        .assign(month_num=lambda x: x["month"].str[:-1].map(int))
        .sort_values("month_num")
        .assign(diff=lambda x: x["temperature"].diff())
        .dropna(subset=["temperature"])
        .filter(["month", "temperature","diff"])
    )

(
    df.astype({"temperature":int})
    .groupby("province")
    .apply(f)
    .reset_index(level=0)
    .sort_index()
)
```

---