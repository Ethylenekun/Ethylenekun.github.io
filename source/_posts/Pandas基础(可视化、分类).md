---
title: Pandas基础(可视化、分类)
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas基础(可视化、分类)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
abbrlink: 1a029631
---
# 可视化

## [plot 绘图](https://gairuo.com/p/pandas-plot)

> 默认以index为x轴，以index对应数据为y轴，也可通过x,y参数指定

- kind：表示绘图的类型，默认为line，折线图

- - line：折线图
  - bar/barh：柱状图（条形图），纵向/横向
  - pie：饼状图
  - hist：直方图（数值频率分布）
  - box：箱型图
  - area：区域图（面积图）
  - scatter：散点图
  - hexbin：蜂巢图

```python
# 绘图默认以索引为x轴,值为y轴
ts = pd.Series(np.random.randn(20), index=pd.date_range("1/1/2000", periods=20))
ts.plot.line()

df = pd.DataFrame(np.random.randn(20, 2), columns=["B", "C"]).cumsum()
df["A"] = pd.Series(list(range(len(df))))
df.plot(x="A", y=[*"BC"])
```

![plot图片](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20260127085143688.png)

```python
ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))
df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index, columns=list("ABCD"))
df = df.cumsum()
print(df)
# 图1：其中A列用左Y轴标注，B列用右Y轴标注，二者共用一个X轴
df.A.plot()  # 对A列作图，同理可对行做图
df.B.plot(secondary_y=True)  # 设置第二个y轴（右y轴）
```

![image-20260127092042861](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20260127092042861.png)

```python
ax = df.plot(secondary_y=["A", "B"])  # 定义column A B使用右Y轴。
# ax（axes）可以理解为子图，也可以理解成对黑板进行切分，每一个板块就是一个axes
ax.set_ylabel("CD scale")  # 主y轴标签
ax.right_ax.set_ylabel("AB scale")  # 第二y轴标签
ax.legend(loc="upper left")  # 设置图例的位置
ax.right_ax.legend(loc="upper right")  # 设置第二图例的位置
```

![image-20260127094713696](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20260127094713696.png)

```python
# 设置图形的颜色
df.A.plot(color='red')
df.B.plot(color='blue')
df.C.plot(color='yellow')
```

> 条形图bar

```python
df2 = pd.DataFrame(np.random.rand(10, 4), columns=["a", "b", "c", "d"])

df2.plot.bar() # 生成条形图

df2.plot(kind="bar",stacked=True).legend(loc="upper right") # 生成堆积条形图,并将图里显示在右上角

df2.plot(kind="barh") # 生成水平条形图
```

---

## [matplotlib pyplot 绘图功能](https://gairuo.com/p/python-matplotlib-pyplot)

```python
import matplotlib.pyplot as plt

plt.rcParams["font.family"] = ["SimHei"]  # 用来正常显示中文标签
plt.rcParams["axes.unicode_minus"] = False  # 用来正常显示负号
```

### 折线图plot

```python
import matplotlib.pyplot as plt

# 设置字体
from matplotlib import rcParams
rcParams["font.family"] = "SimHei"
rcParams["font.size"] = "12"

plt.figure(figsize = (10,5)) # 设置画布大小

# 添加标题
plt.title("销售趋势",color="red",fontsize=18)

months = [f"{_}月" for _ in "1234"]
sales = [100,150,80,130]

# 设置X/Y轴,图例名称,线条颜色,线条粗细,线条样式(- 实线, -- 虚线, -.),数据点类型(. 点, o 圆形,D 菱形, *)
plt.plot(months,sales,label="产品A",color="orange",linewidth=3,linestyle="--",marker="o")

# 添加网格线
plt.grid(axis="y",alpha=0.3,color="blue",linestyle="--")

# 设置坐标轴刻度字体大小/旋转角度
plt.xticks(rotation=30,fontsize=12)
plt.yticks(rotation=90,fontsize=12)

# 添加坐标轴标签
plt.xlabel("月份",fontsize=16)
plt.ylabel("销量(万元)",fontsize=16)
 
# 设置y轴起始范围
plt.ylim(0,200)

# 显示数据标记
for x,y in zip(months,sales):
  plt.text(x,y+1,str(y),ha="center",va="bottom",fontsize=10)

# 添加图例
plt.legend(loc="upper left")

plt.show()
```

### 柱状图bar(h)

```python
plt.bar(subjects,scores,label="学生A",color="orange",width=0.3) # width控制柱体宽度

plt.barh(subjects,scores,label="学生A",color="orange",height=0.4) # 条形图,height控制柱体宽度
```

### 饼图pie

```python
letters = [*"ABCD"]
values = [10,20,5,15]

explode = [0.1,0,0,0] # 爆炸离开中心的距离

plt.pie(
  values,
  explode=explode,
  labels=letters,
  autopct="%.1f%%", # 占比为1个小数位的百分数
  startangle=90, # 初始角度
  wedgeprops={"width":0.6}, # 圆环宽度
  pctdistance=0.7, # 百分比的位置
  shadow=True, # 添加阴影效果
) 

plt.text(0,0,"总计:\n100%",ha="center") # 圆环中心设置文字
```

### 散点图scatter

```python
plt.scatter(x,y,color="blue",alpha=0.5,s=10) # s代表点的大小
```

### 箱型图boxplot

箱子的中线代表中位数

箱子的上边缘和下边缘代表上四分位数(0.75)和下四分位数(0.25)

箱子外侧的上下延伸线端点位置代表极端值的阈值，上端点为上四分位数加上1.5 倍的上、下四分位数之差，记作w_high ，下端点为下四分位数减去1.5 倍的上、下四分位数之差，记作w_low

极端异常值会用空心小球标记

![image-20260127154425397](https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/image-20260127154425397.png)

```python
data = {
  "语文":np.random.randint(60,100,10),
  "数学":np.random.randint(60,100,10),
  "英语":np.random.randint(60,100,10),
}

plt.boxplot(data.values(),tick_labels=data.keys())
```

> 多个图表绘制

```python
f1 = plt.subplot(2,2,1)
f1.plot(months,sales)

f2 = plt.subplot(2,2,2)
f2.bar(months,sales)

f3 = plt.subplot(2, 2, 3)
f3.barh(months, sales)

f4 = plt.subplot(2, 2, 4)
f4.scatter(months, sales)
```

---

# 分类数据

## [分类数据创建](https://gairuo.com/p/pandas-categorical-creation)

```python
# Series
s = pd.Series(["a", "b", "c", "a"], dtype="category")
s
'''
0    a
1    b
2    c
3    a
dtype: category
Categories (3, object): [a, b, c]
'''

# 导入CategoricalDtype，转换分类类型
pd.Series([1, 2, 3]).astype(CategoricalDtype([3, 2, 1], ordered=True))
```

>  `pd.Categorical`

```python
raw_cat = pd.Categorical(["a", "b", "c", "a"],
                         categories=["b", "c", "d"],
                         ordered=False)
s = pd.Series(raw_cat)
'''
Out[12]: 
0    NaN
1      b
2      c
3    NaN
dtype: category
Categories (3, object): [b, c, d]
'''
```

## [分类数据的使用](https://gairuo.com/p/pandas-working-categories)

```python
ser = df['team'].astype("category")

# 类别本身
ser.cat.categories # Index(['A', 'B', 'C', 'D', 'E'], dtype='object')
# 类别顺序
ser.cat.ordered # False

# 类似于`pd.factorize`中`sort=True`的情况
fact = pd.factorize(df['team'],sort=True)
(fact[0]==ser.cat.codes.values).all(),(fact[1]==ser.cat.categories).all()
# [True,True]

# 返回序列对应类别的序号
ser.cat.codes
```

> 追加/删除/更改类别

```python
s = pd.Series(["a", "b", "c", "a"], dtype="category")

s.cat.add_categories(["d"])

s.cat.remove_categories(["c"]) # 删除后对应的类别变为空值

# 删除未使用的类别
s.cat.remove_unused_categories()

# 设置类别
s.cat.set_categories([*"abc"])

# 重命名类别
ser.cat.rename_categories({"E":'e'})
```

> 有序类别

```python
# 排序
ser.cat.reorder_categories([*"EDCBA"],ordered=True) # 必须是由当前序列中无序类别元素构成的列表，不能增加新的类别，也不能缺少原来的类别

# 设置无序类别
ser.cat.as_unordered()

# 设置有序类别
ser.cat.as_ordered()

# 排序后可进行大小比较
ser.cat.reorder_categories([*"EDCBA"],ordered=True)>="A"
```

> **[分类数据操作](https://gairuo.com/p/pandas-categorical-operations)**

**对比**

- 相等性（`==`和`!=`）与长度与分类数据相同的类似列表的对象（列表，序列，数组等）进行比较
- 当 `ordered == True` 并且类别相同时，分类数据与另一个分类系列的所有比较（==，！=，>，> =，<和<=）
- 分类数据与标量(**标量必须在分类数据中**)的所有比较

```python
cat = pd.Series([1, 2, 3]).astype(CategoricalDtype([3, 2, 1], ordered=True))
cat_base = pd.Series([2, 2, 2]).astype(CategoricalDtype([3, 2, 1], ordered=True)）
cat_base2 = pd.Series([2, 2, 2]).astype(CategoricalDtype(ordered=True)）
                                        
# 与具有相同类别和顺序的分类比较或与标量比较
cat > cat_base
'''
0     True
1    False
2    False
dtype: bool
'''

cat > 2
'''
0     True
1    False
2    False
dtype: bool
'''

# 相等比较可用于具有相同长度和标量的任何类似列表的对象
cat == cat_base
'''
0    False
1     True
2    False
dtype: bool
'''
cat == np.array([1, 2, 3])
'''
0    True
1    True
2    True
dtype: bool
'''
cat == 2
'''
0    False
1     True
2    False
dtype: bool
'''
```

**操作**

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "c"],
                             categories=["c", "a", "b", "d"]))

s.value_counts() # 不存在数据也会显示
'''
c    2
b    1
a    1
d    0
dtype: int64
'''

s.groupby(s.values).count() # 不存在数据也会显示
```

---

# 习题链接

[案例](https://gairuo.com/p/pandas-example)

[练习题](https://gairuo.com/p/pandas-exercise)