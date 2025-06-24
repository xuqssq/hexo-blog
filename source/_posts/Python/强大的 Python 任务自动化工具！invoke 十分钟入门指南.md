---
title: '强大的 Python 任务自动化工具！invoke 十分钟入门指南[转自公众号Python猫]'
toc: false
tags:
  - Python
abbrlink: 22812
date: 2020-05-02 09:46:55
categories:
description:
---

我们继续聊聊 Python 任务自动化的话题。

nox 的作者在去年的 Pycon US 上，做了一场题为《Break the Cycle: Three excellent Python tools to automate repetitive tasks》的分享（B站观看地址：https://b23.tv/av86640235），她介绍了三个任务自动化工具：tox、nox 和 invoke，本文的话题正好就是最后的 invoke。

![img](https://gitee.com/djgzs_admin/ArticleImg/raw/master/2020/05/02/2020/05/02/20200502094851)

## 1、invoke 可以做什么？

invoke 是从著名的远程部署工具 Fabric 中分离出来的，它与 paramiko 一起是 Fabric 的两大最核心的基础组件。

除了作为命令行工具，它专注于“任务执行”（task execution），可以标注和组织任务，并通过 CLI（command-line interface，即命令行界面） 和 shell 命令来执行任务。

同样是任务自动化工具，invoke 与我们之前介绍过的 tox/nox 在侧重点上有所不同：

- tox/nox 主要是在打包、测试、持续集成等方面的自动化（当然它们能做的还不止于此）
- invoke 则更具普遍性，可以用在任何需要“执行任务”的场景，可以是无相关性的任务组，也可以是有顺序依赖的分步骤的工作流

invoke 在 Github 上有 2.7K star，十分受欢迎，接下来我们看看它如何使用？

## 2、怎么使用 invoke？

首先，安装很简单：`pip install invoke`。

其次，简单使用时有以下要素：

- 任务文件。创建一个 tasks.py 文件。
- @task 装饰器。在一个函数上添加 @task 装饰器，即可将该函数标记为一个任务，接受 invoke 的调度管理。
- 上下文参数。给被装饰的函数添加一个上下文参数（context argument），注意它必须作为第一个参数，而命名按约定可以是`c` 或`ctx` 或`context` 。
- 命令行执行。在命令行中执行`invoke --list` 来查看所有任务，运行`invoke xxx` 来执行名为 xxx 的任务。命令行中的“invoke”可以简写成“inv”。

以下是一个简单的示例：

```python
# 文件名：tasks.py
from invoke import task

@task
def hello(c):
    print("Hello world!")

@task
def greet(c, name):
    c.run(f"echo {name}加油!")
```

在上述代码中，我们定义了两个任务：

- ”hello“任务调用了 Python 内置的 print 函数，会打印一个字符串“Hello world!”
- “greet”任务调用了上下文参数的 run() 方法，可以执行 shell 命令，同时本例中还可以接收一个参数。在 shell 命令中，echo 可理解成打印，所以这也是一个打印任务，会打印出“xxx加油！”（xxx 是我们传的参数）

以上代码写在 tasks.py 文件中，首先导入装饰器 `from invoke import task`，@task 装饰器可以不带参数，也可以带参数（参见下一节），被它装饰了的函数就是一个任务。

上下文参数（即上例的“c”）必须要显式地指明，如果缺少这个参数，执行时会抛出异常：“TypeError: Tasks must have an initial Context argument!”

然后在 tasks.py 文件的同级目录中，打开命令行窗口，执行命令。如果执行的位置找不到这个任务文件，则会报错：“Can't find any collection named 'tasks'!”

正常情况下，通过执行`inv --list` 或者`inv -l` ，可以看到所有任务的列表（按字母表顺序排序）：

```python
>>> inv -l
Available tasks:

  greet
  hello
```

我们依次执行这两个任务，其中传参时可以默认按位置参数传参，也可以指定关键字传参。结果是：

```python
>>> inv hello
Hello world!
>>> inv greet 武汉
武汉加油!
>>> inv greet --name="武汉"
武汉加油！
```

缺少传参时，报错：'greet' did not receive required positional arguments: 'name'；多余传参时，报错：No idea what '???' is!

## 3、 如何用好 invoke？

介绍完 invoke 的简单用法，我们知道了它所需的几项要素，也大致知道了它的使用步骤，接下来是它的其它用法。

### 3.1 添加帮助信息

在上例中，“inv -l”只能看到任务名称，缺少必要的辅助信息，为了加强可读性，我们可以这样写：

```python
@task(help={'name': 'A param for test'})
def greet(c, name):
    """
    A test for shell command.
    Second line.
    """
    c.run(f"echo {name}加油!")
```

其中，文档字符串的第一行内容会作为摘录，在“inv -l”的查询结果中展示，而且完整的内容与 @task 的 help 内容，会对应在“inv --help”中展示：

```python
>>> inv -l
Available tasks:

  greet   A test for shell command.
>>> inv --help greet
Usage: inv[oke] [--core-opts] greet [--options] [other tasks here ...]

Docstring:
  A test for shell command.
  Second line.

Options:
  -n STRING, --name=STRING   A param for test
```

### 3.2 任务的分解与组合

通常一个大任务可以被分解成一组小任务，反过来，一系列的小任务也可能被串连成一个大任务。在对任务作分解、抽象与组合时，这里有两种思路：

- 对内分解，对外统一：只定义一个 @task 的任务，作为总体的任务入口，实际的处理逻辑可以抽象成多个方法，但是外部不感知到它们
- 多点呈现，单点汇总：定义多个 @task 的任务，外部可以感知并分别调用它们，同时将有关联的任务组合起来，调用某个任务时，也执行其它相关联的任务

第一种思路很容易理解，实现与使用都很简单，但是其缺点是缺少灵活性，难于单独执行其中的某个/些子任务。适用于相对独立的单个任务，通常也不需要 invoke 就能做到（使用 invoke 的好处是，拥有命令行的支持）。

第二种思路更加灵活，既方便单一任务的执行，也方便多任务的组合执行。实际上，这种场景才是 invoke 发挥最大价值的场景。

那么，invoke 如何实现分步任务的组合呢？可以在 @task 装饰器的“pre”与“post”参数中指定，分别表示前置任务与后置任务：

```python
@task
def clean(c):
    c.run("echo clean")

@task
def message(c):
    c.run("echo message")

@task(pre=[clean], post=[message])
def build(c):
    c.run("echo build")
```

clean 与 message 任务作为子任务，可以单独调用，也可以作为 build 任务的前置与后置任务而组合使用：

```
>>> inv clean
clean
>>> inv message
message
>>> inv build
clean
build
message
```

这两个参数是列表类型，即可设置多个任务。另外，在默认情况下，@task 装饰器的位置参数会被视为前置任务，接着上述代码，我们写一个：

```python
@task(clean, message)
def test(c):
    c.run("echo test")
```

然后执行，会发现两个参数都被视为了前置任务：

```python
>>> inv test
clean
message
test
```

### 3.3 模块的拆分与整合

如果要管理很多相对独立的大型任务，或者需要多个团队分别维护各自的任务，那么，就有必要对 tasks.py 作拆分与整合。

例如，现在有多份 tasks.py，彼此是相对完整而独立的任务模块，不方便把所有内容都放在一个文件中，那么，如何有效地把它们整合起来管理呢？

invoke 提供了这方面的支持。首先，只能保留一份名为“tasks.py”的文件，其次，在该文件中导入其它改名后的任务文件，最后，使用 invoke 的 Collection 类把它们关联起来。

我们把本文中第一个示例文件改名为 task1.py，并新建一个 tasks.py 文件，内容如下：

```python
# 文件名：tasks.py
from invoke import Collection, task
import task1

@task
def deploy(c):
    c.run("echo deploy")

namespace = Collection(task1, deploy)
```

每个 py 文件拥有独立的命名空间，而在此处，我们用 Collection 可以创建出一个新的命名空间，从而实现对所有任务的统一管理。效果如下：

```python
>>> inv -l
Available tasks:

  deploy
  task1.greet
  task1.hello
>>> inv deploy
deploy
>>> inv task1.hello
Hello world!
>>> inv task1.greet 武汉
武汉加油!
```

关于不同任务模块的导入、嵌套、混合、起别名等内容，还有不少细节，请查阅官方文档了解。

### 3.4 交互式操作

某些任务可能需要交互式的输入，例如要求输入“y”，按回车键后才会继续执行。如果在任务执行期间需要人工参与，那自动化任务的能力将大打折扣。

invoke 提供了在程序运行期的监控能力，可以监听`stdout` 和`stderr` ，并支持在`stdin` 中输入必要的信息。

例如，假设某个任务（excitable-program）在执行时会提示“Are you ready? [y/n]”，只有输入了“y”并按下回车键，才会执行后续的操作。

那么，在代码中指定 responses 参数的内容，只要监听到匹配信息，程序会自动执行相应的操作：

```python
responses = {r"Are you ready? \[y/n\] ": "y\n"}
ctx.run("excitable-program", responses=responses)
```

responses 是字典类型，键值对分别为监听内容及其回应内容。需注意，键值会被视为正则表达式，所以像本例中的方括号就要先转义。

### 3.5 作为命令行工具库

Python 中有不少好用的命令行工具库，比如标准库中的`argparse`、Flask 作者开源的`click` 与谷歌开源的`fire` 等等，而 invoke 也可以作为命令行工具库使用。

（PS：有位 Prodesire 同学写了“Python 命令行之旅”的系列文章，详细介绍了其它几个命令行工具库的用法，我在公众号“Python猫”里转载过大部分，感兴趣的同学可查看历史文章。）

事实上，Fabric 项目最初把 invoke 分离成独立的库，就是想让它承担解析命令行与执行子命令的任务。所以，除了作为自动化任务管理工具，invoke 也可以被用于开发命令行工具。

官方文档中给出了一个示例，我们可以了解到它的基本用法。

假设我们要开发一个 tester 工具，让用户`pip install tester` 安装，而此工具提供两个执行命令：`tester unit` 和`tester intergration` 。

这两个子命令需要在 tasks.py 文件中定义：

```python
# tasks.py
from invoke import task

@task
def unit(c):
    print("Running unit tests!")

@task
def integration(c):
    print("Running integration tests!")
```

然后在程序入口文件中引入它：

```python
# main.py
from invoke import Collection, Program
from tester import tasks

program = Program(namespace=Collection.from_module(tasks), version='0.1.0')
```

最后在打包文件中声明入口函数：

```python
# setup.py
setup(
    name='tester',
    version='0.1.0',
    packages=['tester'],
    install_requires=['invoke'],
    entry_points={
        'console_scripts': ['tester = tester.main:program.run']
    }
)
```

如此打包发行的库，就是一个功能齐全的命令行工具了：

```python
$ tester --version
Tester 0.1.0
$ tester --help
Usage: tester [--core-opts] <subcommand> [--subcommand-opts] ...

Core options:
  ... core options here, minus task-related ones ...

Subcommands:
  unit
  integration

$ tester --list
No idea what '--list' is!
$ tester unit
Running unit tests!
```

上手容易，开箱即用，invoke 不失为一款可以考虑的命令行工具库。更多详细用法，请查阅文档 。

## 4、小结

invoke 作为从 Fabric 项目中分离出来的独立项目，它自身具备一些完整而强大的功能，除了可用于开发命令行工具，它还是著名的任务自动化工具。

本文介绍了它的基础用法与 5 个方面的中级内容，相信读者们会对它产生一定的了解。invoke 的官方文档十分详尽，限于篇幅，本文不再详细展开，若感兴趣，请自行查阅文档哦。

![音符](https://gitee.com/djgzs_admin/ArticleImg/raw/master/2020/05/02/2020/05/02/20200502094755)

**作者简介：**豌豆花下猫，生于广东毕业于武大，现苏漂程序员，有一些极客思维，也有一些人文情怀，有一些温度，还有一些态度。