---
title: Swarm集群系统搭建和管理技巧
tags:
  - 集群
  - Swarm
  - Docker
categories:
  - Docker
toc: true
toc_number: true
top_img: >-
  https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/photo-1668009358366-a09c5ab0da81.webp
cover: >-
  https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/photo-1668009358366-a09c5ab0da81.webp
abbrlink: 31580
date: 2022-11-24 20:32:52
updated:
keywords:
description:
comments:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---

# Swarm集群系统搭建和管理技巧

## Portainer管理面板

&emsp;&emsp;针对Portainer面板这里不过详细介绍，目前Docker单节点、Swarm集群的管理面板中，没有其他面板能够媲美它了。安装也是一键搞定，所有上手使用非常轻松。[Introduction - Portainer Documentation](https://docs.portainer.io/start/intro)，如果不是服务器配置特别低，建议学习和自建使用可以安装上，可视化操作还是比较方便的。

![portainer登录页面](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/Snipaste_2022-11-24_20-40-26.webp)

![集群节点选择页面](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/Snipaste_2022-11-24_20-43-21.webp)

![集群管理页面](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/Snipaste_2022-11-24_20-43-46.webp)

## Swarm常用命令

### 初始化Swarm集群

&emsp;&emsp;Swarm集群需要开放`2377``7946``4789`这三个端口进行集群通信，特殊的主机商如阿里云文档说明`4789`网络作为常规的UDP通信端口，不提供给用户使用。如果出现通信异常和跨主机网络异常，需要检查这些因素。

```sh
docker swarm init --advertise-addr 公网IP
```

### 添加节点

&emsp;&emsp;添加节点的时候，最好<font>附带上指定IP的参数</font>`--listen-addr IP地址`，部分主机商多层网络比较复杂，自动获取的IP不是公网IP，而是内网的IP，导致端口即使是开放的也无法正常连接Overlay网络。

#### 添加manager节点

```sh
docker swarm join-token manager # 获取添加命令
```

#### 添加worker节点

```sh
docker swarm join-token worker # 获取添加命令
```

### 解散集群和节点主动脱离

```sh
docker swarm leave -f
```

&emsp;&emsp;如果需要离开集群，可以在对应节点执行上面的命令。

## 集群运维技巧

### Overlay网络连接不上的问题

&emsp;&emsp;Overlay网络不通，主要是两个原因：

1. 端口未开放，无法正常通信。
2. 加入节点时未指明节点IP，出现节点IP无数据交换。

### 节点无响应

Swarm集群稳定性不足，重启的节点脱离后，会在节点记录之前的节点信息，重新加入节点却被认为是一个新节点，之前的集群信息未清除，导致无响应，节点异常。执行下面指令清除Swarm集群信息后，再加入集群。

```sh
docker swarm leave -f
docker network rm docker_gwbridge
systemctl stop docker
rm -rf /var/lib/docker/swarm
systemctl start docker
```

另外一种情况是管理节点异常，例如3个管理节点，出现一个管理节点掉线，两个管理节点无法选举Leader节点，如果节点比较多，建议`5节点`或`7节点`，根据Raft算法，管理节点最好是基数，并且建议最多`7节点。`此时可以使用`docker node demote 节点ID`让一台管理节点变成worker节点。然后再恢复3管理节点。

### 内存不足的问题

由于集群的长期使用，动态更新，系统内残留无用的日志、镜像、容器、Cache会堆在内存中，一般当docker反馈这个信息的时候，已经是宿主机内存分配完了`df -hl`可以看到磁盘没剩多少空间了。`docker system df`可以查看docker使用的内存空间。`docker system prune -a`命令能够清空docker无用的文件，这个命令是最干净的。如果需要针对性的清除，可以清除镜像即可，例如`docker image prune -a`。

### Service约束的使用

可以通过`constraints`参数限制服务启动在哪个节点，一般都是添加对应的标签进行`==`、`!=`判断。例如

```yaml
deploy:
  mode: global
  placement:
    constraints:
      - node.labels.role!=web
```

