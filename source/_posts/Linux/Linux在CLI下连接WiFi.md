---
title: Linux在CLI下连接WiFi
tags:
  - Linux
toc: false
abbrlink: 58864
date: 2018-06-28 19:29:00
categories:
description:
---
![](https://ws1.sinaimg.cn/large/e3bf8736gy1fyqcfaxx0dj22801hcb2e.jpg)
<!--more-->

系统安装好后，有线与线连接都可以使用， 切换联网方式执行要使用“`ifdown 对应的网卡名称`”或者“`ifup 对应的网卡名称`”这两条命令即可。

使用 nmcli命令，查看各网卡的状态。得知无线网卡已经被驱动起来，并且已经纳入NetworkManager的管理。

```shell
[root@localhost ~]# nmcli dev status
DEVICE      TYPE      STATE   CONNECTION   
wlp3s0b1    wifi      连接的  Daoji_Studio 
virbr0      bridge    连接的  virbr0       
enp2s0      ethernet  不可用  --           
lo          loopback  未托管  --           
virbr0-nic  tun       未托管  --           
```

这是我的WiFi连接情况 

如果无线网卡没有被纳入NetworkManager的管理，则可以安装”NetworkManager-wifi” ，命令如下。 

1.设置NetworkManager自动启动 


```shell
chkconfig NetworkManager on
```

2.安装NetworkManager-wifi 


```shell
yum -y install NetworkManager-wifi
```

运行这条命令后重启centos。进入系统后，打开NetworkManager，设置好WiFi后，就可以连接到WiFi了。
上述步骤进行完以后，若WIFI仍然没开启，或者启动后无法自动连接WiFi，可以这样开启WIFI: 

```shell
nmcli r wifi on // 开启WIFI
nmcli dev wifi //扫描可用WIFI
nmcli dev wifi connect password // 连接WIFI
```
