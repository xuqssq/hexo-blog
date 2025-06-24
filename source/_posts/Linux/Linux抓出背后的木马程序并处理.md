---
title: Linux抓出背后的木马程序并处理
tags:
  - Linux
toc: false
abbrlink: 19424
date: 2018-07-17 08:49:00
categories:
description:
---
![](https://tva1.sinaimg.com/large/e3bf8736gy1fyqcp6wozhj225r1fux6s.jpg)
<!--more-->

### ·通过top命令查看进程情况

    top  
![](https://yqfile.alicdn.com/3fc0dc0046bf64d22c788b86063c20378756710b.png)  	
获取`pid`后可以用`kill [PID]`来关闭进程	
​	

### ·通过进程查询异常程序所在目录

通过执行`ll /proc/$PID/exe`，($PID即进程ID)可获得异常进程的目录	

![](https://yqfile.alicdn.com/249ac3353c4e4dc8482f1df07e1b059cb420d78a.png)	

此程序一般是由计划任务产生的，Linux系统中默认创建了计划任务后会在/var/spool/cron目录下创建对应用户的计划任务脚本，执行ls /var/spool/cron 查询一下系统中是否有异常的计划任务脚本程序。 
可以看到，在此目录下有1个root的计划任务脚本和一个异常的目录crontabs（默认情况下不会有此目录，用户创建计划任务也不会产生此目录） 

![](https://yqfile.alicdn.com/add7471c149fe18e39d9048bc65caace21701c25.png)  

查看脚本内容，有一个每隔10分钟便会通过curl下载执行的脚本程序（crontabs目录下为同样内容的计划任务）  

![](https://yqfile.alicdn.com/d89bc571d8ba01a1f9eab99af0112674c67196aa.png)

手动将脚本内容下载到本地，脚本内容如下

![](https://yqfile.alicdn.com/38ba3d1db70e742993f16bd8c0cf155422fc58a8.png)

#### 分析此脚本，主要进行了如下修改

1、创建了上述查看到的两个计划任务脚本  
2、创建了密钥认证文件，导入到了/root/.ssh目录下（当前脚本的密钥文件名是KHK75NEOiq，此名称可能会有所变化，要根据具体情况进行核实）  
3、修改ssh配置文件允许了root远程登录，允许了密钥认证，修改默认的密钥认证文件名  
4、重启了sshd服务使配置生效  
5、创建了伪装程序ntp，并运行了ntp程序   
6、查询系统中是否有正常运行的计划任务，杀死正在运行的计划任务进程。  

#### 【处理方法】

根据以上分析，提供以下处理方法  
1、删除计划任务脚本中异常配置项，如果当前系统之前并未配置过计划任务，可以直接执行rm -rf /var/spool/cron/* 情况计划脚本目录即可。  
2、删除黑客创建的密钥认证文件，如果当前系统之前并未配置过密钥认证，可以直接执行rm -rf /root/.ssh/* 清空认证存放目录即可。如果有配置过密钥认证，需要删除指定的黑客创建的认证文件即可，当前脚本的密钥文件名是KHK75NEOiq，此名称可能会有所变化，要根据具体情况进行核实。  
3、修复ssh配置项，根据个人需求进行修改，一般默认脚本中进行修改的PermitRootLogin、RSAAuthentication、PubkeyAuthentication为开启状态，需要修改的是密钥认证文件名，建议修改成默认值AuthorizedKeysFile .ssh/authorized_keys即可。修改完成后重启sshd服务，使配置生效即可。  
4、删除黑客创建的伪装程序ntp  
执行ls /etc/init.d/可以看到系统中是由对应的伪装程序的  

![](https://yqfile.alicdn.com/72eede7da7d1bfb0c1c32f8c1e99993408a0cb70.png)

通过chkconfig --list ntp 可以看到此程序默认设置的是开机自动启动

![](https://yqfile.alicdn.com/ade809df30ef7d688b551b2b0b559149a5800571.png)

如果此程序不进行清除，即使删除了minerd程序并且杀死了对应的进程，过一会系统还会重新创建minerd程序，并产生新的进程  
查询一下当前系统中是否有ntp进程，可以看到ntp进程是通过/usr/sbin/ntp程序产生，因此需要把对应的执行程序也进行删除

![](https://yqfile.alicdn.com/1199403d7e19bd225e5c65525a6ed8ce6bc1100f.png)

总结一下删除伪装程序的操作步骤  

```shell
kill -9 $PID 杀死查询到的ntp进程  
rm -rf /etc/init.d/ntp  
rm -rf /usr/sbin/ntp （此路径要根据具体的查询数据确定，实际情况可能会有所变化）  
```



### ·删除异常程序并关闭异常进程

根据之前的查询minerd程序所在路径为`/opt`，在执行的脚本中同时也在`/opt`目录下创建了一个`KHK75NEOiq33`的程序文件

因此要删除这两个文件，执行`rm -rf KHK75NEOiq33 minerd` 即可

![](https://yqfile.alicdn.com/091056648e466a2f53301188b13de33b86039709.png)

`kill -9 $PID` 杀死对应的进程ID 

通过ps命令查询一下minerd对应的进程详细情况
​	ps -aux|grep minerd
![](https://yqfile.alicdn.com/c92e1ee20ba41c253bb6c3786919e5439e469edc.png)

### ·修改SSH端口以及采用（字母+符号+数字=密码）的方式登录

### ·适当时可以考虑一下网上的防爆破SSH脚本或程序