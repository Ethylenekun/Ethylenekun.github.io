---
title: Pandas前置知识
tags:
  - Python
  - Pandas
categories: Pandas
description: Pandas前置知识(UV/Python/Numpy)
cover: 'https://gcore.jsdelivr.net/gh/Ethylenekun/images/img/Pandas.png'
swiper_index: 6
abbrlink: a35d5512
---
# uv配置虚拟环境

[功能介绍](https://uv.doczh.com/getting-started/features/)

> 更改uv镜像源，修改`pyproject.toml`文件

```toml
[[tool.uv.index]]
default = true
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
```

## Python管理

[python安装](https://uv.doczh.com/guides/install-python/)

- `uv python install 3.12`: 安装 Python 版本
- `uv python list`: 查看可用 Python 版本
- `uv python uninstall`: 卸载 Python 版本

## 项目开发

[项目开发](https://uv.doczh.com/guides/projects/)

- `uv init`: 创建新 Python 项目
- `uv venv`: 创建`.venv`虚拟环境,`uv venv --python 3.11.6`指定版本
- `uv add`: 为项目添加依赖
- `uv add -r requirements.txt`：从 requirements.txt 安装
- `uv remove`: 从项目移除依赖
- `uv lock --upgrade-package requests`: 升级包
- `uv sync`: 同步项目依赖到环境
- `uv run`: 在项目环境中运行命令
- `uv tree`: 查看项目依赖树

---

# Python前置知识

## 推导式

```python
# 嵌套推导式，优先进行外层循环

# 靠前的为外层循环，靠后的为内层循环
[(color, item) for color in ["red", "blue"] for item in ["pen", "book"]]
# [('red', 'pen'), ('red', 'book'), ('blue', 'pen'), ('blue', 'book')]

# 外层推导式内的迭代为外层循环，内层推导式内的迭代为内层循环
[[(color, item) for color in ["red","blue"]] for item in ["pen", "book"]]
# [[('red', 'pen'), ('blue', 'pen')], [('red', 'book'), ('blue', 'book')]]
```

> 给定3 个二维整数列表L1、L2、L3，它们都是30×20 的列表，即每个列表中包含30 个内层列表，并且每一个内层列表中包含20 个整数。请利用列表推导式，构造一个大小相同的新列表Lnew，其满足任意一个位置的值是L1、L2、L3 相应位置的值的最小值。

```python
import random
random.seed(0)
n = 10000
L1 = [[random.randint(-n,n) for i in range(20)] for j in range(30)]
L2 = [[random.randint(-n,n) for i in range(20)] for j in range(30)]
L3 = [[random.randint(-n,n) for i in range(20)] for j in range(30)]

Lnew = [[min([L1[j][i], L2[j][i], L3[j][i]]) for i in range(20)] for j in range(30)]
```

# Numpy教程

## 特殊矩阵

```python
# 对角矩阵
np.diag([1,2,3,4])
'''
array([[1, 0, 0, 0],
       [0, 2, 0, 0],
       [0, 0, 3, 0],
       [0, 0, 0, 4]])
'''
```

## 变形

> `T` / `transpose` / `swapaxes`

```python
carbon = np.random.rand(360, 180, 3)

carbon.transpose(1,0,2).shape # [180,360,3]
carbon.T.shape # [3,180,360]
carbon.swapaxes(1,2).shape # [360,3,180]
```

> `reshape`

```python
matrix = np.arange(8)

matrix.reshape(2,-1,order="C") # C先填充内部
"""
array([[0, 1, 2, 3],
       [4, 5, 6, 7]])
"""

matrix.reshape(2,-1,order="F") # F先填充外部
"""
array([[0, 2, 4, 6],
       [1, 3, 5, 7]])
"""
```

> 合并 / 拆分变形

```python
# 合并
pop_man = np.random.rand(20, 6, 3)  # 男性（随机生成）
pop_women = np.random.rand(20, 6, 3)  # 女性（随机生成）

np.stack([pop_man,pop_women],axis=1).shape # [20,2,6,3] 合并创建新的维度
np.concatenate([pop_man, pop_women], axis=1).shape # [20,12,3] 合并现有维度

# 拆分
np.split(pop_man,indices_or_sections=3,axis=1)[0].shape # [20,2,3] 均分成三等份
[i.shape for i in np.split(pop_man,indices_or_sections=[1,4],axis=1)] # [(20, 1, 3), (20, 3, 3), (20, 2, 3)] 按照分割点

# 重复
# 一维数组
a = np.arange(3)
np.repeat(a,2) # array([0,0,1,1,2,2])
np.repeat(a,[1,2,3]) # array([0,1,1,2,2,])
# 二维数组
a = np.array([[1,2],[3,4]])
np.repeat(a, repeats=[1,2], axis=1) # [[1,2,2],[3,4,4]]
np.repeat(a, repeats=[1,2], axis=0) # [[1,2],[3,4],[3,4]]
```

> 请使用repeat()函数构造两个10×10 的数组，第一个数组要求第i 行的元素值都为i，第二个数组要求第i 列的元素值都为i

```python
arr = np.arange(1,11)
np.repeat(arr.reshape(10,1),repeats=10,axis=1) # 等效于arr[:,None]
np.repeat(arr.reshape(1,10),repeats=10,axis=0) # 等效于arr[None,:]
```

## 切片

```python
target = np.arange(24).reshape(4, 2, 3)
"""
array([[[ 0,  1,  2],
        [ 3,  4,  5]],

       [[ 6,  7,  8],
        [ 9, 10, 11]],

       [[12, 13, 14],
        [15, 16, 17]],

       [[18, 19, 20],
        [21, 22, 23]]])
"""

# 按具体位置取出
target[[0, 1], [0, 1], [0, 1]]  # array([0, 10])

# 连续多个出现在最初几个维度的全体切片“:”，可以使用...来简写
target[..., 0::2]
```

## 常用函数

```python
# 标准差 / 方差 -> 方差 sum((每个值-平均值)**2) / 值个数 -> 标准差 sqrt(方差)
np.var
np.std

# 最大值/最小值索引 (类似pandas的idxmax/idxmin)
np.argmax
np.argmin
# 返回非0数的索引
a = np.array([0, -5, 0, 1, 3, -1])
np.nonzero(a) # (array([1, 3, 4, 5]),)

# 分位数
np.percentile(arr,50) # 等同于中位数
# 两种方法相同
np.percentile(arr,20)
np.quantile(arr,q = 0.2)

# 数值大小比较 np.less_equal
np.greater([1, 2, 3, 4], 2)  # array([False, False,  True,  True])
np.less([1, 2, 3, 4], 2)  # array([ True, False, False, False])
np.equal([1, 2, 3, 4], 2)  # array([False,  True, False, False])

# 逻辑运算
np.logical_and
np.logical_or
np.logical_not

np.any
np.all

# 条件运算
arr = np.arange(1,11)
np.where(arr % 2 == 0,arr,np.nan) # array([nan,  2., nan,  4., nan,  6., nan,  8., nan, 10.])

arr = np.arange(1,11)
np.select([arr % 2==0,True],[1,2]) # 若为偶数返回1,否则返回2 / 返回的数据类型需与之前一致

# 排序
np.sort

# 去重
np.unique
```

> 二维扩展

```python
my_matrix = np.random.randint(20, 40, 24).reshape(2, 3, 4)# 维度分别代表学校、年级、班级
'''
array([[[21, 25, 37, 31],
[39, 32, 36, 34],
[37, 27, 31, 24]],
[[37, 38, 22, 33],
[30, 33, 26, 33],
[20, 22, 21, 35]]])
'''

my_matrix.sum(axis=(1, 2)) # 把年级和班级的维度聚合起来，只保留学校
# array([374, 350])
```

> 给定一个维度为m×n 的整数数组，请返回一个元素为0 或1 的同维度数组，当且仅当某位置在原数组中的对应元素是原数组同行元素中的最大值时，该位置的元素取1。

```python
matrix = np.random.randint(1,10,(5,5))

np.where(matrix == matrix.max(axis=1).reshape(5,1),1,0)
```

---