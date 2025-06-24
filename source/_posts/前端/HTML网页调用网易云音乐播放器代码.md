---
title: HTML网页调用网易云音乐播放器代码
toc: false
tags:
  - 前端
abbrlink: 2893
date: 2020-05-02 10:22:06
categories:
description:
---

### 表现形式一：单曲播放



**调用代码：**

```html
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=86 src="http://music.163.com/outchain/player?type=2&id=299757&auto=1&height=66"></iframe>
```

**参数说明：**

> **播放器可修改参数：**
> 
> **width=100% #自适应宽度**
> 
> **height=86 #根据自己喜好修改**
> 
> **id=299757 #为歌曲的ID http://music.163.com/#/song?id=299757**
> 
> **auto=0 #0为不自动播放，1为自动播放**

**效果图：**

![img](https://gitee.com/djgzs_admin/ArticleImg/raw/master/2020/05/02/2020/05/02/20200502102248.png)



### 表现形式二：列表播放

**调用代码：*

```html
<iframe src="http://music.163.com/outchain/player?type=0&amp;id=34238509&amp;auto=0&amp;height=430" width="100%" height="450" frameborder="no" marginwidth="0" marginheight="0"></iframe>
```

**参数说明：**

> **播放器可修改参数：**
> 
> **width=100% #自适应宽度**
> 
> **height=450#根据自己喜好修改**
> 
> **id=34238509#为歌曲列表页的ID ,例如：http://music.163.com/#/playlist?id=34238509**
> 
> **auto=0 #0为不自动播放，1为自动播放**

**效果图：**



![img](https://gitee.com/djgzs_admin/ArticleImg/raw/master/2020/05/02/2020/05/02/20200502102327.png)

***fds***

