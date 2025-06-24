---
title: 史上最全python字符串操作指南
toc: false
tags:
  - Python
abbrlink: 50672
date: 2020-04-28 23:16:12
categories:
description:
---

### **字符串的定义**

日常编码中，大家会发现，太多时候我们需要对数据进行处理，而这数据不管是数组、列表、字典，最终都逃不开字符串的处理。
所以今天要来跟大家发散的聊聊**字符串**！
估计很多人看到是将字符串肯定觉得索然无味(老子都会)，可大佬们不妨再往下看看？

![img](https://gitee.com/djgzs_admin/ArticleImg/raw/master/2020/04/28/2020/04/28/20200428221803)

python定义字符、字符串没有java那样的严格，不管是单引号、双引号、甚至是三个单引号和双引号都可以用来定义字符(串)，只要成对出现即可。比如：

```python
# 单个字符
a='a'
# 使用单引号定义字符串
name='Uranus'
# 使用双引号定义字符串
code = "Hello World ..."
# 既然说到了string，怎么能不点开源码看看呢？
class str(object):
    """
    str(object='') -> str
    str(bytes_or_buffer[, encoding[, errors]]) -> str
    Create a new string object from the given object. If encoding or
    errors is specified, then the object must expose a data buffer
    that will be decoded using the given encoding and error handler.
    Otherwise, returns the result of object.__str__() (if defined)
    or repr(object).
    encoding defaults to sys.getdefaultencoding().
    errors defaults to 'strict'.
    """
```

虽然这些不是主要说的，但还是简单提下，三个单引号或者双引号，主要是用来作为文档注释的，请不要拿来定义字符串(虽然这样并不会出现语法错误)。
今天主要说下关于打段的字符串应该如何定义，PEP8有规定，一行代码的长度请勿超过120个字符。那么如果遇到这种情况，该怎么办？

```python
# 不推荐的使用方式：
line = """
Create a new string object from the given object.
If encoding or errors is specified,
then the object must expose a data buffer that will be
decoded using the given encoding and error handler.
"""
# 或者这样
line = "Create a new string object from the given object. " \
       "If encoding or errors is specified," \
       "then the object must expose a data buffer that will be" \
       " decoded using the given encoding and error handler."
# 更好的实现方式：
line = ("Create a new string object from the given object."
        "If encoding or errors is specified,"
        "then the object must expose a data buffer that will be "
        "decoded using the given encoding and error handler."
        )
```



### **字符串中.is()的用法**

**.is\*()**, 既然是is，那么它返回的结果只有两种，True or False
**先来对比一下数字：**

> isdigit()
> True: Unicode数字，byte数字（单字节），全角数字（双字节），罗马数字
> False: 汉字数字
> Error: 无
>
> isdecimal()
> True: Unicode数字，全角数字（双字节）
> False: 罗马数字，汉字数字
> Error: byte数字（单字节）
>
> isnumeric()
> True: Unicode数字，全角数字（双字节），罗马数字，汉字数字
> False: 无
> Error: byte数字（单字节)

总结几个偏门知识点：

```python
a='①②③④⑤'
isdigit()、isnumeric() 为True isdecimal()为False
b='一壹'
isnumeric()会认为是True的哦！
```

**再来看一个等式：**

> isalnum() = isdigit() + isalpha() + isspace()
> isdigit()表示字符串内全部为数字
> isalpha()表示字符串内全部为字符
> isspace()表示字符串有一个或多个空格组成
> isalnum()表示字符串内全部为数字和字符

```python
a='12345'
b='①②③④⑤'
c='abc123'

print(a.isdigit()) # True
print(b.isalpha()) # True
print(c.isalnum()) # True
```

**针对字符串大小写的方法：**

> .isupper() 字符串全部由大写组成
> .islower() 字符串全部由小写组成
> .istitle() 字符串形式为驼峰命名，eg:"Hello World"

以上这些用法去掉is，则变为了对应的字符串转发方法。学一套会两套，买一送一….

最后说一个不带.的is* --- isinstance(obj,type)

> 判断一个object是什么类型…
> type可选类型为：int，float，bool，complex，str，bytes，unicode，list，dict，set，tuple
> 并且type可以为一个原组：isinstance(obj, (str, int))



### **判断字符串中的内容**

.*with() starts ends 不仅支持开头结尾的匹配，还支持start和end两个参数来动态定义字符串的index位置

```python
long_string = "To live is to learn，to learn is to better live"
long_string.startswith('To')
long_string.startswith('li', 3, 5)
long_string.endswith('live')
long_string.endswith('live', 0, 7)
```

同样支持start、end来判断字符串的还有 .find()、.rfind()和 .index()、.rindex()
这两类字符串寻址方法均支持从左到右、从右至左两种寻址方式，不同的是：
find在未找到时，返回-1，而index在未找到时，会抛出ValueError的异常…

```python
long_string.index('live') # 3
long_string.rindex('live') # 42
```



### **字符串的内容变更**

狭义来说使用，字符串的替换使用.replace()即可，那为什么还要单独说呢？因为它有一个可选参数**count**

```python
long_string = "To live is to learn，to learn is to better live"
long_string.count('live') # 2
long_string.replace('live','Live',1)
output:
'To Live is to learn，to learn is to better live'
# 可以看到，第二个live并未进行替换
```

**刚才说了狭义，那么广义呢？**

1. **(l/r)strip()**

   将字符串左、右、两端的特定字符过滤掉，默认为空格…
   strip()要注意的地方是，strip('TolLive') 中的字符并非完整匹配，而是针对每一个字符进行匹配，说起来混，直接上例子：

    ```python
   long_string = "To live is to learn，to learn is to better live"
   long_string.strip('TolLive')
   's to learn，to learn is to better'
    ```

2. **字符串切片**

   字符串的切片分为long_string[start:end;step] start、end区间为左闭右开…这个网上说的太多了，再拉出来详细讲就要挨打了…

   **(l/r)just(width,[fillchar])、center(width, [fillchar])、zfill(width)**
   这些均为填充固定长度的字符，默认使用空格(zfill为左补0，z是zero的意思…),看意思就明白了，不用多讲了….



### **字符串格式化输出**

本来fill和center等可以放在这里，但是他们使用频率和重量级不够格，就丢在上面了。
Python格式化输出分为**两类**，那是在pyton2的时代，即 % 和 format。这两种网上的资料太多了，说的太多显得没逼格…
但，还是要简单说说其中特殊的地方

1. **% 格式化输出：**

   - 如何在%的格式输出中，输出用来看做标记为的%符号呢？使用两个百分号（%%）
   - %(-)(width) width为设置长度，默认左填充空格，添加-号为右填充
   - .width代表字符串截断，保留多少长度的字符串
   - type %s字符串 %d十进制整数  %f小数 …
   - 多个参数是，后面的参数需要使用括号包裹起来

   ```python
   '姓名：%-5s 年龄：%4d 爱好： %.8s' % ('王大锤',30,'python、Java')
   output：
   '姓名：王大锤   年龄：  30 爱好：python、J'
   ```

2. **format格式输出：**

   format在python3开始官方就表示为替换%的输出方式，之所以还保留着%，主要是为了兼容性考虑…

   - 对比%，format使用花括号{}表示变量
   - < > ^ 代表了format的对齐方式

   ```python
   '{:-^40s}'.format('华丽的分割线')
   output:
   '-----------------华丽的分割线-----------------'
   ```



3. **f-string**

   Python3.6的版本更新时，新增了f-string，英文好的可以去看官方解释PEP 498 -- Literal String Interpolation 。
   f-string是字符串引号前以f/F开头，并使用{}标注替换位置的使用形式。
   之所以官方推出f-string，主要是因为它的更高的性能、更强的功能。例子走起：

   ```python
   name = 'Uranus'
   f'Hello,{name}'
   f'Hello,{name.lower()}'
   f'Hello,{name:^10s}'
   f'Hello,{(lambda x: x*2) (name)}'
   
   output:
   'Hello,Uranus'
   'Hello,uranus'
   'Hello,  Uranus  '
   'Hello,UranusUranus'
   ```

