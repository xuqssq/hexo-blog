---
title: alembic教程
tags:
  - alembic
  - Python
  - SQLAlchemy
categories:
  - Python
toc: true
toc_number: true
abbrlink: 56655
date: 2023-03-03 11:14:02
updated:
keywords:
description:
top_img:
comments:
cover:
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

# alembic教程

## 常用命令

### 数据库升级到最新版本

`alembic upgrade head`

### 生成版本文件

`alembic revision --autogenerate -m "操作注释"`

### 降级到最初版本

`base`是代表最初版本号，也可以降级指定版本号`1234567890`

`alembic downgrade base`

### 查询当前版本号

在数据库中有一个alembic_version的字段，表示的是最后一个版本的版本号





```
alembic命令和参数解释：

1. init：创建一个alembic仓库。

2. revision：创建一个新的版本文件。

3. --autogenerate：自动将当前模型的修改，生成迁移脚本。

4. -m：本次迁移做了哪些修改，用户可以指定这个参数，方便回顾。

5. upgrade：将指定版本的迁移文件映射到数据库中，会执行版本文件中的upgrade函数。

如果有多个迁移脚本没有被映射到数据库中，那么会执行多个迁移脚本。

6. [head]：代表最新的迁移脚本的版本号。

7. downgrade：会执行指定版本的迁移文件中的downgrade函数。

8. heads：展示head指向的脚本文件版本号。

9. history：列出所有的迁移版本及其信息。

10. current：展示当前数据库中的版本号。

```

