---
title: Numpy的random函数汇总
date: 2018-06-06 09:54:44
categories:
- Machine Learning
tags:
- Numpy Random
---

## 一、前言
在`python`数据分析的学习和日常应用过程中，经常要用到`numpy`库的随机函数，这些随机函数功能很多，经常会混淆记不住～～，每次使用都要去`Google`，所以我就把常用的随机函数功能汇总在下面了，方便自己查阅。

```
import numpy as np
```

## 二、`numpy.random.rand()`

语法：
```
numpy.random.rand(d0,d1,...,dn)
# rand函数根据给定维度生成[0,1)之间的数据，包含0，不包含1
# dn表格每个维度
# 返回值为指定维度的array
```

例子：
```
np.random.rand(4,2)
# output
array([[0.44966653 0.52431591]
       [0.12117809 0.31425446]
       [0.94938082 0.78223903]
       [0.98802125 0.01795372]])

np.random.rand(4,3,2)
#output
array([[[0.4016697  0.41116077]
        [0.81121991 0.18433693]
        [0.94307831 0.37959419]]

         [[0.11907587 0.97932363]
          [0.20458545 0.73305606]
          [0.62938875 0.99506056]]

         [[0.85168379 0.76133277]
          [0.868105   0.29375361]
          [0.08666713 0.43133631]]

         [[0.93674358 0.50061816]
          [0.58272102 0.40495383]
          [0.94631605 0.51712061]]])

```

## 三、`numpy.random.randn()`返回值具有标准正太分`N(0,1)`

语法：
```
numpy.random.randn(d0,d1,...,dn)
# randn函数返回一个或一组样本，具有标准正态分布。
# dn表格每个维度
# 返回值为指定维度的array
```

例子：
```
np.random.randn() # 当没有参数时，返回单个数据
# output
0.10125281554614712

np.random.randn(2,4)
# output
array([[ 0.88550167  0.94488988  0.68380494 -2.42685228]
 [-0.12320336 -0.21325373  0.07526319 -0.59034683]])

np.random.randn(4,3,2)
# output
array([[[ 1.09663108 -0.58652581]
      [-0.94125426  0.01259422]
      [-0.4189963   0.03315681]]

     [[-0.38388522 -0.46672398]
      [ 0.35335792 -0.92017071]
      [-0.26904716  2.28057168]]

     [[-0.37893325 -0.64560053]
      [ 0.93308775  0.68629363]
      [ 2.6449362   1.40476443]]

     [[ 1.90709637  0.18952578]
      [-0.09167653  1.95480404]
      [-1.55128951 -0.07542736]]])

```

## 四、`numpy.random.randint()`

### 4.1、 `numpy.random.randint()`

语法：
```
numpy.random.randint(low, high=None, size=None, dtype=’l’)

    返回随机整数，范围区间为[low,high），包含low，不包含high
    参数：low为最小值，high为最大值，size为数组维度大小，dtype为数据类型，默认的数据类型是np.int
    high没有填写时，默认生成随机数的范围是[0，low)

```

例子：
```
np.random.randint(1,size=5) # 返回[0,1)之间的整数，所以只有0
# output
array([0, 0, 0, 0, 0])

np.random.randint(1,5) # 返回1个[1,5)时间的随机整数
# output
3

np.random.randint(-5,5,size=(2,2))
# output
array([[-4 -5]
       [-2 -2]])


```

### 4.2、 `numpy.random.random_integers`

语法：
```
numpy.random.random_integers(low, high=None, size=None)

    返回随机整数，范围区间为[low,high]，包含low和high
    参数：low为最小值，high为最大值，size为数组维度大小
    high没有填写时，默认生成随机数的范围是[1，low]

该函数在最新的numpy版本中已被替代，建议使用randint函数
```

## 五、生成`[0,1)`之间的浮点数

语法：
```
    numpy.random.random_sample(size=None)
    numpy.random.random(size=None)
    numpy.random.ranf(size=None)
    numpy.random.sample(size=None)

```

例子：
```
print('-----------random_sample--------------')
print(np.random.random_sample(size=(2,2)))
print('-----------random--------------')
print(np.random.random(size=(2,2)))
print('-----------ranf--------------')
print(np.random.ranf(size=(2,2)))
print('-----------sample--------------')
print(np.random.sample(size=(2,2)))

# output
-----------random_sample--------------
[[0.73845538 0.57461322]
 [0.50090024 0.12809794]]
-----------random--------------
[[0.65293998 0.15751317]
 [0.0745293  0.03874293]]
-----------ranf--------------
[[0.75927288 0.79548232]
 [0.47618339 0.04334719]]
-----------sample--------------
[[0.81010338 0.59291716]
 [0.58648785 0.1767814 ]]

```

## 六、 `numpy.random.choice()`

语法：
```
numpy.random.choice(a, size=None, replace=True, p=None)

    从给定的一维数组中生成随机数
    参数： a为一维数组类似数据或整数；size为数组维度；p为数组中的数据出现的概率
    a为整数时，对应的一维数组为np.arange(a)

```

例子：
```
np.random.choice(5,3)
# output
array([4 4 3])

np.random.choice(5, 3, replace=False)
# 当replace为False时，生成的随机数不能有重复的数值
# output
array([4 2 0])

np.random.choice(5,size=(3,2))
# output
[[0 0]
 [1 3]
 [0 4]]

demo_list = ['lenovo', 'sansumg','moto','xiaomi', 'iphone']
np.random.choice(demo_list,size=(3,3))
# output
array([['sansumg' 'iphone' 'moto']
       ['moto' 'iphone' 'iphone']
       ['xiaomi' 'iphone' 'xiaomi']])

```


    参数p的长度与参数a的长度需要一致；
    参数p为概率，p里的数据之和应为1
```
demo_list = ['lenovo', 'sansumg','moto','xiaomi', 'iphone']
np.random.choice(demo_list,size=(3,3), p=[0.1,0.6,0.1,0.1,0.1])
# output
array([['moto' 'sansumg' 'xiaomi']
       ['lenovo' 'moto' 'sansumg']
       ['sansumg' 'sansumg' 'sansumg']])

```

## 七、`numpy.random.seed()`

语法：

    np.random.seed()的作用：使得随机数据可预测。
    当我们设置相同的seed，每次生成的随机数相同。如果不设置seed，则每次会生成不同的随机数

```
np.random.seed(0)
np.random.rand(5)
# output
array([0.5488135 , 0.71518937, 0.60276338, 0.54488318, 0.4236548 ])

np.random.seed(1676)
np.random.rand(5)
# output
array([0.39983389, 0.29426895, 0.89541728, 0.71807369, 0.3531823 ])

np.random.seed(1676)
np.random.rand(5)
# output
array([0.39983389, 0.29426895, 0.89541728, 0.71807369, 0.3531823 ])

```

## 八、结束语

嗯？～～