---
title: Python random 模块小记
date: 2017-04-21 14:33:32
tags: python, random
categories: python
---

## Python random 模块小记

### 概述

该模块为多种分布实现伪随机数生成器。

对于整数，在某范围内均匀选择。 对于序列，对随机元素均匀选择，通过产生随机排列就地生成列表以及用于随机抽取而不进行替换的功能。

### Bookkeeping 函数

#### random.seed(a=None, version=2)
> 初始化随机数字产生器
> 如果省略a或None，则使用当前系统时间。 如果操作系统提供随机源，则使用它们而不是系统时间（有关可用性的详细信息，请参阅os.urandom（）函数）。
> 如果a是int, 则直接使用
> 使用版本2（默认值），str，bytes或bytearray对象将被转换为int，并使用其所有位。
> 使用版本1（提供用于从旧版本的Python重现随机序列），str和字节的算法产生较窄的种子范围。

<!-- more -->

#### random.getstate()
> 返回捕获当前生成器内部状态的对象。这个对象可以通过setstate()恢复状态

#### random.setstate(state)
> 状态应该是从之前的函数调用getstate()获得的，而setstate()将生成器的内部状态恢复到当时调用getstate()的状态。

### Functions for integers

#### random.randrange(stop) / random.randrange(start, stop, [, step])
> 返回一个从range(start, stop, step)随机抽取的元素. 这个相当于choic(range(start, stop, step)), 但实际上并不构建range对象
> 位置参数与range()函数相匹配. 不应该使用关键词参数,因为函数可能会以不寻常的方式使用他们

#### random.randint(a, b)
> 返回一个随机的数字N, a <= N <= b. 函数相当于randrange(a, b + 1)

### Functions for sequences

#### random.choice(seq)
> 从非空序列seq返回随机元素。 如果seq为空，则引发IndexError。

#### random.shuffle(x[, random])
> 随机打乱序列x, 会改变x本身.

#### random.sample(population, k)
> 返回从序列或集合中选择的K长度的list, list中元素唯一。 用于随机抽样,但不会替换本身
> 返回一个新列表，其中包含来自population的元素，同时保持原始population不变。 所得到的列表处于选择顺序，使得所有子片段也将是有效的随机样本。 这允许抽奖优胜者（样本）被划分为大奖和第二名获奖者（下级）。
> population不需要是可散列的或独特的。 如果population包含重复元素，则样本中每一个元素都可能会出现。
> 要从整数范围中选择一个样本，请使用range()对象作为参数。 这对于从很大的population进行抽样来说，特别快速，空间有效：sample(range(1000000), 60)
> 如果样本大小大于总体大小，则会引发ValueError。

### 分布函数
函数参数以分布方程中的相应变量命名，如通用数学实践中所使用的; 大多数这些方程可以在任何统计文本中找到。

#### random.random()
> 返回范围[0.0，1.0)中的下一个随机浮点数。

#### random.uniform(a, b)
> 返回一个随机浮点数N，若a <= b, 则返回的N会a <= N <= b, 若b < a, b <= N <= a
> 根据等式a +（b-a）* random（）中的浮点舍入，终点值b可以包括也可以不包括在范围内。

还有一些分布函数没有整理出来,如果需要可以查阅文档.

### 应用

#### 基本应用
``` bash
>>> random.random()                      # Random float x, 0.0 <= x < 1.0
0.37444887175646646

>>> random.uniform(1, 10)                # Random float x, 1.0 <= x < 10.0
1.1800146073117523

>>> random.randrange(10)                 # Integer from 0 to 9
7

>>> random.randrange(0, 101, 2)          # Even integer from 0 to 100
26

>>> random.choice('abcdefghij')          # Single random element
'c'

>>> items = [1, 2, 3, 4, 5, 6, 7]
>>> random.shuffle(items)
>>> items
[7, 3, 2, 5, 6, 4, 1]

>>> random.sample([1, 2, 3, 4, 5],  3)   # Three samples without replacement
[4, 1, 5]
```

#### 进阶
一个常见的任务是产生一个带加权比率的random.choice()

如果权重是小的整数比率，一个简单的技术是建立重复的样本population：
``` bash
>>> weighted_choices = [('Red', 3), ('Blue', 2), ('Yellow', 1), ('Green', 4)]
>>> population = [val for val, cnt in weighted_choices for i in range(cnt)]
>>> population
['Red', 'Red', 'Red', 'Blue', 'Blue', 'Yellow', 'Green', 'Green', 'Green', 'Green']
>>> random.choice(population)
'Red'
```

更一般的方法是使用itertools.accumulate()权重排列在累积分布中，然后使用bisect.bisect()定位随机值：
``` bash
>>> choices, weights = zip(*weighted_choices)
>>> choices
('Red', 'Blue', 'Yellow', 'Green')
>>> weights
(3, 2, 1, 4)
>>> import itertools
>>> cumdist = list(itertools.accumulate(weights))
>>> cumdist
[3, 5, 6, 10]
>>> x = random.random() * cumdist[-1]
>>> x
7.904737262461592
>>> import bisect
>>> choices[bisect.bisect(cumdist, x)]
'Green'
```

### 后记
目前对于Python随机用的不多,不过既然用到了,我还是整理了一下官方的文档,加油.