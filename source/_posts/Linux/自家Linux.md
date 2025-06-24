---
title: 自家Linux
tags:
  - Linux
toc: false
abbrlink: 57501
date: 2018-07-17 08:49:00
categories:
description:
---
![](https://ws1.sinaimg.cn/large/e3bf8736gy1fyqgf924gsj21z81bhx6u.jpg)
<!--more-->
### #centos7配置第一篇
##GUI界面锁屏解除

    #设置->隐私->锁屏和通知  都关闭

##CLI界面合盖休眠解除

```shell
vi /etc/systemd/logind.conf
```
#HandlePowerKey       //按下电源键后的行为，默认 `=poweroff`

#HandleSuspendKey     //按下挂起键后的行为，默认 `=suspend`

#HandleHibernateKey   //按下休眠键的行为，默认 `=hibernate`

#HandleLidSwitch      //合上笔记本盖后的行为，默认 `=suspend`，应该改为 `=lock`，并且在文件中去除前面的`#`

运行配置文件使其生效 ###


```shell
systemctl restart systemd-logind
```

