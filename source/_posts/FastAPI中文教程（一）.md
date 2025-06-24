---
title: FastAPI中文教程（一）
tags:
  - fastapi
  - pydantic
  - Python
  - SQLAlchemy
categories:
  - Python
  - fastapi
toc: true
toc_number: true
abbrlink: 180
date: 2023-03-12 22:13:58
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

# FastAPI系列教程（一）

## 教程资源

[ChristopherGS的英文教程](https://christophergs.com/tutorials/ultimate-fastapi-tutorial-pt-1-hello-world/)

[pyb4430/full-stack-fastapi-postgresql: Full stack, modern web application generator. Using FastAPI, PostgreSQL as database, Docker, automatic HTTPS and more. (github.com)](https://github.com/pyb4430/full-stack-fastapi-postgresql)

> 上面的教程是根据FastAPI全栈生成的项目模板进行讲解的，对于有一定基础、熟悉容器化技术、熟悉部署细节的同学，可以快速的掌握FastAPI的用法，进行生产项目的开发。
>
> [pyb4430/full-stack-fastapi-postgresql]仓库是fork的官方并进行积极维护的一个仓库，至该文章编写时间2023/3/12官方仓库对于旧模块失效出现多处bug仍未进行修复，推荐根据该全栈仓库学习全栈部署。

> 官方文档是最好的学习教程，FastAPI作者Tangolo比较忙，经常开坑，FastAPI的文档更新也不是很完善，部分章节内容没有进行更新（例如没有详细讲解依赖注入的异步写法，或者大多代码使用的都是非异步写法，SQLalchemy也没详细讲解异步写法），这部分虽然不是很重要，可以在模块官方和Python进阶教程里面学习到，但是对于初学者就不是很友好，无法系统的学习，会感觉到混乱。

## 前言

至文章编写时间，目前Python Web框架从性能上来讲，FastAPI基于Starlette开发，处于Python Web开发框架的最优选。据调查（根据目前数个Web[框架基准测试排行网站](https://www.techempower.com/benchmarks/#section=data-r21)），从基准测试来说，FastAPI能达到同时13000并发请求时，同为Python框架的Django、Flask只能达到1k-2k。但是如果要同Golang的GoFrame、Gin、Echo、Iris等框架轻松实现10W+请求数相比，不足为论。如果用Golang，非常推荐GoFrame框架，文档齐全、社区活跃、性能优秀，并且是国人开源并积极维护。

![](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/20230312224716.png)![](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/20230312224755.png)



> 注：该调查虽然不严谨，但是有大概的参考价值，Web开发细节复杂，性能高低可根据开发者经验和技术进行优化，但是框架的基准测试则比较稳定。

> Django-Ninja框架目前也备受关注，经过试用，确实是能够与普通的django程序比有极大的性能提升。但是考虑到仍然基于django框架开发，数据库对异步的支持不高，社区开源项目不丰富，称不上是一个优秀的选择。



![](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/20230312225035.png)

![](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/20230312225257.png)

Swoole框架是由C语言编写的高性能请求框架，单论请求数来说，能够实现同比30w+的并发请求数。但是真正运营于web开发，需要编写路由功能、请求解析、数据校验、数据库交互等，如laravel-swoole也才16000并发请求数。

所以同学们讨论这个问题不能太片面，像本文重点FastAPI web开发框架是基于Starlette请求框架开发的，Startlette请求框架又是基于Uvicorn异步框架（Python高性能ASGI异步协议库）开发的，尽快其性能再强大，web开发中需要编写路由功能、请求解析、数据校验、数据库交互、安全验证等，最终也会和FastAPI框架差不多，甚至不如。而直接使用FastAPI框架，能够快速开发web网站，其自带Pydantic数据校验（Python数据校验和解析库，备受关注，非常推荐开发自己的项目时使用）等功能，有不少的开源项目和示例，[github issue](https://github.com/tiangolo/fastapi/issues)容易查找你开发时遇见的常见问题。

## 安装

官方项目使用Poetry这个Python环境管理工具，所以官方项目里面是没有requirement.txt文件的，笔者初步尝试，感觉不错，后面讲解一下这个工具的简单使用。[官方文档地址](https://fastapi.tiangolo.com/zh/)有部分中文翻译章节，可以参考。

首先安装`FastAPI`模块：

```shell
pip install fastapi
```

还需要安装ASGI服务器，笔者这里使用[Uvicorn](https://www.uvicorn.org/)，官方文档中 [Hypercorn](https://gitlab.com/pgjones/hypercorn)也是支持的。

```sh
pip install uvicorn[standard] # 最小安装
```

这样就能编写最简单的FastAPI项目了。

## 示例

```python
from typing import Union

from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}

import uvicorn
uvicorn.run(app)
```

运行代码可以简单访问并获取响应了，`http://127.0.0.1:8000`可以获取返回内容。

## API文档

FastAPI自带API文档，**通过路由映射自动生成OpenAPI文件**（一种接口协议规格文件），并提供两种UI界面展示，分别是**Swagger UI和ReDoc**。对于后端API开发来说，能够更加便捷的进行接口调试（一旦尝试过这种体验就回不去了，类似DRF框架），对于前端来说，能够提供一份严格的API文档，方便同步开发（为了接口规范，最好还是开发之前设计好架构，商议确定接口路径和参数，开发中的API总是和结果不一样的）。

## 特性

### 校验和解析

FastAPI最为突出的还是**请求参数的校验和解析**，在Python中数据类型是动态的、自动推断的，我们不否认这是一个很好的特性，但是在Web开发中我们需要协调多个模块，通常这些模块是由不同人员或组完成的，开发语言和规范很难统一，尤其在微服务领域里，这些甚至来自不同组织来完成。因为在后端中能够进行严格参数校验和快速解析，这是非常重要的功能（确保参数类型错误，参数不齐这些低级错误不是来源自身，怼人有自信）。这个功能在Springboot、GoFrame、Laravel中都存在，但是在Python传统web框架Django、Flask中却没有，django有名的扩展插件DRF也不过具有参数校验功能，但是却不具备高效的解析和清晰且便捷的文档。

Type Hints——[Python PEP 484提案](https://peps.python.org/pep-0484/)和Pydantic这个库的出现才改变这个局面，加上Pycharm对Type Hints的完美支持（其实对于生成器和旧模块还没很好支持）使得开发效率有进一步的提升。他强大之处只有体验过才能理解，只需要像静态语言一样对属性名添加类型注释，既可以对输入参数校验，对输出参数进行解析。

没有开发经验的同学可能没法很好理解，例如Json数据结构目前仍是前后端最流行数据结构体，但是Json数据中对浮点数不支持，因此传输过程Python需要将浮点数float转成字符str，再传到前端，从前端接收同样需要由str转float，如果前端传输数据不规范，传过来的整数非字符类型，而是数字类型，类型转换就会出错。又或者是前端传输参数缺少的情况。

虽然通过繁琐且复杂的校验和解析能够避免这些情况（为了推卸责任，不能不写😘），但是同Pydantic（Cython实现）实现的功能对比，还是不要重复造轮子了，有那时间同学们不想打游戏吗？或者学习？

#### 在IDE中

- 自动补全
- 类型检查

#### 数据校验

- 在校验失败时自动生成清晰的错误信息
- 对多层嵌套的 JSON 对象依然执行校验

#### 转换 来自网络请求的输入数据为 Python 数据类型。包括以下数据

- JSON
- 路径参数
- 查询参数
- Cookies
- 请求头
- 表单
- 文件

#### 转换 输出的数据：转换 Python 数据类型为供网络传输的 JSON 数据

- 转换 Python 基础类型 （`str`、 `int`、 `float`、 `bool`、 `list` 等）
- `datetime` 对象
- `UUID` 对象
- 数据库模型
- ......以及更多其他类型

#### 自动生成的交互式 API 文档，包括两种可选的用户界面

- Swagger UI
- ReDoc

### 依赖注入

Python是动态语言，轻易修改运行时变量对象，即依赖注入。不同于静态语言预编译程序有严格检查，且编译和运行时处于不同内容，Python则能够轻松修改。FastAPI框架自身提供了依赖注入的方式实现数据库连接获取，身份认证，Session和Cookie解析等中间件功能。这里不推荐传统中间件实现方式的理由是，中间件会在每个请求时都执行，消耗内存高，对于不需要的请求增加多余代码运行，降低效率，而依赖注入可以在需要使用的请求上执行，提供很好的解耦，更加优雅。

依赖注入的实现原理和代码，笔者会在后面的文章中介绍，此处不做讲解。

### 安全性及身份验证

集成了安全性和身份认证。杜绝数据库或者数据模型的渗透风险。

OpenAPI 中定义的安全模式，包括：

- HTTP 基本认证。
- **OAuth2** (也使用 **JWT tokens**)。在 [OAuth2 with JWT](https://fastapi.tiangolo.com/zh/tutorial/security/oauth2-jwt/)查看教程。
- API 密钥，在:
  - 请求头。
  - 查询参数。
  - Cookies, 等等。

加上来自 Starlette（包括 **session cookie**）的所有安全特性。

所有的这些都是可复用的工具和组件，可以轻松与你的系统，数据仓库，关系型以及 NoSQL 数据库等等集成。

