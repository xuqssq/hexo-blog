---
title: Android studio如何用GIF图作为背景
tags:
  - Android
toc: false
abbrlink: 53646
date: 2018-08-08 21:07:00
categories:
description:
---
![](https://ws1.sinaimg.cn/large/e3bf8736ly1fypyrvuet2j22lo1qgnpj.jpg)

<!--more-->

### 引用第三方库

1、先将你需要的GIF进行压缩，不然有可能会内存溢出

2、将你的GIF放到drawable当中

3、引入GIF依赖

```
//引入GIF背景动态图实现依赖
compile 'pl.droidsonroids.gif:android-gif-drawable:1.1.+'
```


4、添加自定义GIF控件

```
<pl.droidsonroids.gif.GifImageView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/lutos_background" /> 
```

5、完成