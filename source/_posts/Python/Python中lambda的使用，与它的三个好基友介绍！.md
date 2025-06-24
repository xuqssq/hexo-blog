---
title: Python中lambda的使用，与它的三个好基友介绍！
toc: false
tags:
  - Python
abbrlink: 18713
date: 2020-04-28 21:56:39
categories:
description:
---

### 匿名函数lambda

除了def语句，python还提供了一种生成函数对象的表达式形式。由于它与LISP语言中的一个工具类似，所以称为lambda。
就像def一样，这个表达式创建了一个之后能够调用的函数，但是它返回一个函数而不是将这个函数赋值给一个变量。这些就是lambda叫做匿名函数的原因。实际上，他常常以一种行内进行函数定义的方式使用，或者用作推迟执行一些代码。
lambda的一般形式是关键字lambda之后跟着一个或多个参数（与一个def头部内用括号括起来的参数列表类似），紧跟着是一个**冒号**，之后是表达式

> lambda arg1，arg2,argn:expression using arguments

由lambda表达式所返回的函数对象与由def创建并复制后的函数对象工作起来是完全一致的，但lambda有一些不同之处，让其扮演特定的角色时更有用：

###### lambda是一个表达式，而不是一个语句

因为这一点，lambda可以出现在python语法不允许def出现的地方。
此外，作为一个表达式，lambda返回一个值（一个新的函数），可以选择性的赋值给一个变量
相反，def语句总是得在头部将一个新的函数赋值给一个变量，而不是将这个函数作为结果返回。

###### lambda的主题是单个表达式，而不是一个代码块

这个lambda的主题简单的就好像放在def主体return语句中的代码一样。
简单的将结果写成一个顺畅的表达式，而不是明确的返回。
但由于它仅限于表达式，故lambda通常要比def功能少…你仅能够在lambda主体中封装有限的逻辑进去，因为他是一个为编写简单函数而设计的。
除了上述这些差别，def和lambda都能过做同样种类的工作

def与lambda的相同用法



```python
x = lambda x, y, z: x + y + z
x(2, 3, 4)
>>> 9

y = (lambda a='hello', b='world': a + b)
y(b='清风')
>>> 'hello清风'
```

### 为什么使用lambda

看过上面的两个小例子，很多人会说这个和def没什么差别，我们又为什么要使用lambda呢？通常来说，lambda起到一种函数的速写作用，允许在使用的代码内嵌一个函数的定义，他完全是可选的(是可以使用def代替他们)，但是在你仅需要切入一段可执行代码的情况下，它会带来一个更简洁的书写效果。

lambda通常用来编写跳转表，也就是行为的列表或者字典，能够按照需求执行操作，比如：

```python
l = [lambda x: x ** 2, lambda x: x ** 3, lambda x: x ** 4]

for f in l:
    print(f(2))
>>> 4
>>> 8
>>> 16
print(l[0](3))
>>> 9
```

当需要把小段的可执行代码编写进def语句从语法上不能实现的地方是，lambda表达式作为def的一种速写来说，是最为有用的，如果上面的代码用def编写，则变为：

```python
def f1(x):
    return x ** 2

def f2(x):
    return x ** 3

def f3(x):
    return x ** 4

l = [f1, f2, f3]

for f in l:
    print(f(2))
print(l[0](3))
```

实际上，我们可以用python中的字典或者其他的数据结构来构建更多种类的行为表，从而做同样的事情。

### lambda中实现if-else

Python中具备的单行表达式：**`if a:b else c`**语法在lambda中同样适用：

```python
lower = lambda x,y:x if x<y else y
lower(4,5)
>>> 4
```

看了半天，大家可能也并未觉得lambda在python中到底比def优越与便利在哪里，那么说到lambda，就必须要提及三个函数**`map、filter、reduce`**，当你接触了这三个函数，那么你才能感受到lambda真实的方便之处

### map 函数

程序对列表或者其他序列常常要做的一件事就是对每个元素进行一个操作，并把其结果集合起来。
python提供了一个工具map，它会对一个序列对象中的每一个元素应用该的函数，并返回一个包含了所有函数调用结果的列表。
举个栗子，我们有一个列表，需要将列表的每一个字段+10，我们该如何操作？

```python
list_show = [1, 2, 3, 4]
# 方式1
new_list_show = []
for i in list_show:
    new_list_show.append(i + 10)

print(new_list_show)

# 方式2
def adds(x):
    return x + 10

print(list(map(adds, list_show)))

# 更优雅的方式3：
print(list(map(lambda x: x + 10, list_show)))
```

看看上面三个实现方式，你觉得那种更加Pythonic？
eg:需要注意一点，map在python3中是一个可迭代对象，引入需要使用列表调用来使它生成所有的结果用于显示，python2不必如此。
当然map的阐述函数，不仅仅支持自己编写的，同样也支持python自带的多种函数，比如：

```python
list_show = [1, -2, 3, -4, 5, -6]
print(list(map(abs, list_show)))
>>> [1, 2, 3, 4, 5, 6]
```



### filter函数

filter通过字面意思，大家就知道它的用处了，用于数据的过滤操作，它也是lambda的一个好基友，举个栗子。
我们需要过滤0-9中，能被2整除的数字组成一个列表，我们该如何操作？只需要一行代码：

```python
print(list(filter(lambda x: x % 2 == 0, range(10))))
>>> [0, 2, 4, 6, 8]
```

没错，filter就是这么的简单实用….

### reduce的妙用

> reduce在python2中是一个简单的函数，但在python3中它责备收录与functools中。
> 它接收一个迭代器来处理并返回一个单个的结果。

```python
list_show = [1, 2, 3, 4]
print(reduce(lambda x, y: x + y, list_show))
>>> 10
print(reduce(lambda x, y: x * y, list_show))
>>> 24
```

