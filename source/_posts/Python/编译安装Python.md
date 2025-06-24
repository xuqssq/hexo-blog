---
title: 编译安装Python
toc: true
tags:
  - Python
  - Linux
description: Linux系统安装Python最新版一般都是要编译安装
abbrlink: 19621
date: 2020-04-28 19:58:47
categories:
---

### 准备：

- 安装`gcc` `g++`编译器和`make` :

  ```shell
  sudo apt install gcc g++ make
  ```

- 安装依赖：

    ```shell
    sudo apt install zlib1g-dev libssl-dev libffi-dev libsqlite3-dev uuid-dev libbz2-dev libreadline-dev liblzma-dev libncurses5-dev libmysqlclient-dev
    ```

- 解压Python安装包

    ```shell
    wget https://www.python.org/ftp/python/3.7.6/Python-3.7.6.tgz
    ```

### **开始安装**：

- `cd`到刚刚解压的Python路径中，然后运行 `./configure`命令:

    ```shell
    cd Python-3.7.6
    ./configure --enable-optimizations
    ```

- 运行以下命令进行安装：

  ```shell
  make
  sudo make install
  ```

- 查看安装版本：

    ```shell
    python3 -V
    ```

### **安装最新版**`pip` :

- 下载

  ```shell
  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  sudo python3 get-pip.py -i  https://mirrors.aliyun.com/pypi/simple/
  ```

- 查看安装版本：

  ```shell
  pip -V
  # 或者
  pip3 -V
  ```

### **修改pypi镜像源**：

- 清华：

  ```shell
  pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
  ```

- 阿里云：

  ```shell
  pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/
  ```

- 安装第三方库时可能还需要：

  ```shell
  sudo apt install python3-dev    # 可以暂时不安装
  ```