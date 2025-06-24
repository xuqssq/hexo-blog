---
title: 手动编译安装nginx
toc: false
tags:
  - Python
abbrlink: 29613
date: 2020-05-03 15:42:42
categories:
description:
---

### 前言

&emsp;这里采用的是`CentOS 7`系统演示。安装工具有些差别，但是原理流程是一样的。

### 更新系统软件

```shell
yum update -y
```



### 安装编译工具

```shell
yum install gcc -y
```



### 安装pcre、pcre-devel

&emsp;pcre是一个perl库，包括perl兼容的正则表达式库，nginx的http模块使用pcre来解析正则表达式，所以需要安装pcre库。

```shell
yum install -y pcre pcre-devel
```



### 安装zlib

&emsp;zlib库提供了很多种压缩和解压缩方式nginx使用zlib对http包的内容进行gzip，所以需要安装。

```shell
yum install -y zlib zlib-devel
```



### 安装openssl

&emsp;OpenSSL是http通信加密的库。

```shell
yum install -y openssl openssl-devel
```



### 下载官方源码

```shell
wget http://nginx.org/download/nginx-1.9.9.tar.gz
```



### 解压

```shell
tar -zxvf  nginx-1.9.9.tar.gz
```



### 配置编译安装

&emsp;在解压目录下执行。

```shell
./configure
 
make
 
make install
```

