---
title: 一日一技：Numpy进阶之排序小技巧
toc: false
tags:
  - Python
abbrlink: 14916
date: 2020-05-02 09:44:29
categories:
description:
---

Numpy提供了大量用数组操作的函数，其中不乏常见的排序函数。

这里讲下`numpy.sort`、`numpy.argsort`、`numpy.lexsort`三种排序函数的用法。

### 1、如何对数组元素进行快速排序？

使用`numpy.sort`函数可以对数组进行排序，并返回排序好的数组。

使用方法：

```
numpy.sort(a, axis=-1, kind=None, order=None)
```

参数：

- a : 要排序的数组；
- axis ：按什么轴进行排序，默认按最后一个轴进行排序；
- kind ：排序方法，默认是快速排序；
- order :  当数组定义了字段属性时，可以按照某个属性进行排序；

```
import numpy as np
# 创建一个一维数组
x1 = np.array([1,8,2,4])
x1
'''
一维数组：
array([1, 8, 2, 4])
'''
# 排序
np.sort(x1)
'''
输出：
array([1, 2, 4, 8])
'''
import numpy as np
# 创建一个二维数组
x2 = np.array([[1,8,2,4],[4,5,1,3]])
x2
'''
二维数组：
array([[1, 8, 2, 4],
       [4, 5, 1, 3]])
'''
# 默认按最后一个轴排序，这里按行排序
np.sort(x2)
'''
输出：
array([[1, 2, 4, 8],
       [1, 3, 4, 5]])
'''
# 轴设为0，即按列排序
np.sort(x2,axis=0)
'''
输出：
array([[1, 5, 1, 3],
       [4, 8, 2, 4]])
'''
```

下面试下按照字段属性进行排序，需要用到`order`参数。

```
import numpy as np
# 这是一个名字、身高、年龄的数组
# 先给各字段配置属性类型
dtype = [('Name', 'S10'), ('Height', float), ('Age', int)]
# 各字段值
values = [('Li', 1.8, 41), ('Wang', 1.9, 38),('Duan', 1.7, 38)]
# 创建数组
a = np.array(values, dtype=dtype)
a
'''
数组：
array([(b'Li', 1.8, 41), (b'Wang', 1.9, 38), (b'Duan', 1.7, 38)],
      dtype=[('Name', 'S10'), ('Height', '<f8'), ('Age', '<i4')])
'''
# 按照属性Height进行排序,此时参数为字符串          
np.sort(a, order='Height')     
'''
输出：
array([(b'Duan', 1.7, 38), (b'Li', 1.8, 41), (b'Wang', 1.9, 38)],
      dtype=[('Name', 'S10'), ('Height', '<f8'), ('Age', '<i4')])
'''
# 先按照属性Age排序,如果Age相等，再按照Height排序，此时参数为列表     
np.sort(a, order=['Age', 'Height']) 
'''
输出：
array([(b'Duan', 1.7, 38), (b'Wang', 1.9, 38), (b'Li', 1.8, 41)],
      dtype=[('Name', 'S10'), ('Height', '<f8'), ('Age', '<i4')])
'''
```

### 2、如何获取数组元素排序后的索引？

`numpy.argsort`函数用于将数组排序后，返回数组元素从小到大依次排序的所有元素索引。

使用方法（和sort类似）：

```
numpy.argsort(a, axis=-1, kind=None, order=None)
```

参数：

- a : 要排序的数组；
- axis ：按什么轴进行排序，默认按最后一个轴进行排序；
- kind ：排序方法，默认是快速排序；
- order :  当数组定义了字段属性时，可以按照某个属性进行排序；

```
import numpy as np
# 创建一维数组
x = np.array([3, 1, 2])
'''
数组：
array([3, 1, 2])
'''
# 获取排序后的索引
np.argsort(x)
'''
输出：
array([1, 2, 0], dtype=int64)
'''
import numpy as np
# 创建二维数组
x2 = np.array([[0, 3], [2, 2]])
'''
数组：
array([[0, 3],
       [2, 2]])
'''
# 默认按照最后一个轴进行排序，即行排序
# 获取排序后的索引
np.argsort(x2)
'''
输出：
array([[0, 1],
       [0, 1]], dtype=int64)
'''
```

按字段属性进行排序，并获取索引。

```
# 先给各字段配置属性类型
dtype = [('name', str), ('age', int)]
# 值
values = [('Anna', 28), ('Bob', 27),('Brown',21)]
# 创建数组
x = np.array(values, dtype=dtype)
x
'''
数组：
array([('', 28), ('', 27), ('', 21)],
      dtype=[('name', '<U'), ('age', '<i4')])
'''
# 先按照属性name排序,如果name相等，再按照age排序
np.argsort(x,order=['name','age'])
'''
输出：
array([2, 1, 0], dtype=int64)
'''
```

### 3、如何按多条件进行排序？

> 这里举一个应用场景：
>
> 小升初考试，重点班录取学生按照总成绩录取。
>
> 在总成绩相同时，数学成绩高的优先录取，在总成绩和数学成绩都相同时，按照英语成绩录取…… 
>
> 这里，总成绩排在电子表格的最后一列，数学成绩在倒数第二列，英语成绩在倒数第三列。

`numpy.lexsort`函数用于按照多个条件（键）进行排序，返回排序后索引。

使用方法：

```
numpy.lexsort(keys, axis=-1)
```

参数：

- keys ：序列或元组，要排序的不同的列；
- axis ：沿指定轴进行排序；

说明： 

使用键序列执行间接稳定排序。

给定多个排序键（可以将其解释为电子表格中的列），lexsort返回一个整数索引数组，该数组描述按多个列排序的顺序。

序列中的最后一个键用于主排序顺序，倒数第二个键用于辅助排序顺序，依此类推。

keys参数必须是可以转换为相同形状的数组的对象序列。

如果为keys参数提供了2D数组，则将其行解释为排序键，并根据最后一行，倒数第二行等进行排序。

```
import numpy as np
# 英语成绩
eng = [90,85,95,80]
# 数学成绩
math = [80,95,90,85]
# 总成绩
total = [170,170,185,165]
# 排序，获取索引
np.lexsort((eng,math,total))
'''
先按总成绩total进行排序，
再按数学成绩math进行排序，
最后按英语成绩进行排序。
可以看到total里有两个170，
这时候就按下一级math排序，
最后获取排序后的索引
输出：
array([3, 0, 1, 2], dtype=int64)
'''
# 也可以直接传入数组
score = np.array([[90,85,95,80],[80,95,90,85],[170,170,185,165]])
np.lexsort(score)
'''
输出：
array([3, 0, 1, 2], dtype=int64)
'''
```



### 参考资料

[1]Numpy文档: *https://numpy.org/devdocs/index.html*

[2]Numpy教程: *https://www.runoob.com/numpy/numpy-tutorial.html*