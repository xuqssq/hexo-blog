---
title: Android Studio 3.4.0利用Modelu生成Jar依赖包
toc: false
tags:
  - Android
abbrlink: 16151
date: 2019-01-23 12:28:40
categories:
description:
---

![](https://ws1.sinaimg.cn/large/e3bf8736gy1fzgf0b6s51j21900u01kx.jpg)

<!--more-->

1、新建一个AS项目(不详细介绍)
2、点击File->New->New Module
3、选择Android Library点next
4、点击Finish


5、将要打成Jar包的类放入新建的library目录下


6、打开新建的library下的`build.gradle`文件最后加入如下代码
```
task makejar(type: Copy){
    //删除原来的jar包
    delete 'libs/test.jar'

  //从该目录下拷贝生成的jar包(各版本AndroidStudio目录可能不一样最好自己检查一遍目录)
    from('build/intermediates/intermediate-jars/release/')
     
    //拷贝到该目录
    into('libs')
     
    include('classes.jar')
    //命名文件为test.jar
    rename('classes.jar','test.jar')
}
makejar.dependsOn(build)
```


7、在命令行输入`gradlew makejar`按回车
8、等待一段时间显示`build successful`就可以了
9、把目录显示切换成`project`模式在下图目录`libs`中就能找到了