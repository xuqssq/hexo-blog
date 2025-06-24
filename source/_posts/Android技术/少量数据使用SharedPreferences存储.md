---
title: 少量数据使用SharedPreferences存储
toc: false
tags:
  - Android
abbrlink: 42127
date: 2019-01-23 09:42:30
categories:
description:
---

![](https://ws1.sinaimg.cn/large/e3bf8736gy1fzgecpyfuvj21900u01kz.jpg)

<!--more-->

### 简介

&emsp;&emsp;SharedPreferences是使用**键值对**来存储数据的，所以每当保存和取出一条数据，需要给这条数据提供一个对应的键。而且SharedPreferences是支持多种不同的数据类型存储，也就是当存入的数据类型是什么样，取出来就是什么样的。SharedPreferences进行数据持久化要比使用文件方便的多。

### 将数据存进SharedPreferences

&emsp;&emsp;要使用SharedPreferences存储数据，首先要获取SharedPreferences对象，Android提供的三种获取SharedPreferences对象的方法。

1. Context类中的`getSharedPreferences()`方法

   此方法接收两个参数。

   > ①用于指定SharedPreferences文件名称  //不存在则创建一个，存放`/data/data/<packge name>/shared_prefs/`目录下
   >
   > ②用于指定操作模式  //目前只有`MODE_PRIVATE`可选，传入0值相同，其他模式均被抛弃

2. Activity类中的`getPreferences()`方法

   该方法只接收操作模式`MODE_PRIVATE`，该方法被调用的时候自动以当前活动的类名作为SharedPerferences的文件名。

3. PreferenceManager类中的`getDefaultSharedPreferences()`方法

   这是一个静态的方法，它接收一个Context参数，并自动使用当前程序的包名为前缀命名SharedPerferences文件。

获取SharedPerferences对象后，就能向SharedPreferences中添加数据了，分3步实现

> ⑴调用SharedPerferences对象的edit()方法来获取一个SharedPerferences.Editor对象。
>
> ⑵向SharedPerferences.Editor对象中添加数据，比如添加一个布尔型数据就使用putBoolean()方法，添加一个字符串就使用putString()方法，以此类推。
>
> ⑶调用apply()方法将添加的数据提交，从而完成数据存储操作。

### 从SharedPreferences中取出数据

&emsp;&emsp;SharedPerferences对象提供了`getString（）`等方法对数据进行读取，get方法均要传入两个参数，第一个为键，第二个为默认值，例如`getSharedPreferences("data",MODE_PRIVATE).getString("name","")`、`getSharedPreferences("data",MODE_PRIVATE).getBoolean("eated",false)`。读取的SharedPreferences对象创建方法，应该和存储的SharedPreferences对象一样，否则会找不到SharedPreferences文件。



注：`Ctrl + P`可以查看一个方法的参数类型，规范的代码会带有注释说明。自动补全在类名后加点可以查看该类可以被外部引用的方法。`Ctrl+点击`进去代码查看也可以。