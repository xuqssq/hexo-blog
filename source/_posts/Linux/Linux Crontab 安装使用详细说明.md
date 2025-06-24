---
title: Linux Crontab 安装使用详细说明
tags:
  - Linux
toc: false
abbrlink: 56521
date: 2018-06-28 21:30:00
categories:
description:
---
![](https://tva1.sinaimg.com/large/e3bf8736ly1fypzllrsn1j21250pfhdt.jpg)

<!--more-->

> &emsp;&emsp;crontab命令常见于Unix和Linux的操作系统之中，用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。通常，crontab储存的指令被守护进程激活。crond 常常在后台运行，每一分钟检查是否有预定的作业需要执行。这类作业一般称为cron jobs。

<br>

### 一、安装 ###

```shell
yum -y install vixie-cron
yum -y install crontabs
```

#### 说明：<br>

vixie-cron 软件包是 cron 的主程序；<br>crontabs 软件包是用来安装、卸装、或列举用来驱动 cron 守护进程的表格的程序。

### 二、配置 ###

cron 是 linux 的内置服务，但它不自动起来，可以用以下的方法启动、关闭这个服务： 

```shell
service crond start     //启动服务
service crond stop      //关闭服务
service crond restart   //重启服务
service crond reload    //重新载入配置
service crond status    //查看crontab服务状态
```

 在CentOS系统中加入开机自动启动:  


```shell
chkconfig --level 345 crond on
```

#### 列子： ####

```shell
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
```