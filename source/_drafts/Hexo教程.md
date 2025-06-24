---
title: Hexo教程
date: 2025-02-25 14:03:07
updated: 
tags: 
  - blog
categories:
  - 未分类
keywords:
description:
top_img:
comments:
cover:
toc: true
toc_number: true
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

## 创建文章

```bash
# 基本命令
hexo new "文章标题"

# 使用指定布局模板
hexo new post "文章标题"

# 创建草稿
hexo new draft "文章标题"

# 指定路径创建文章
hexo new post --path posts/2024/article-name "文章标题"
```

## 模板布局

自定义文章模板。在 `scaffolds` 目录下创建或修改模板：

> [!NOTE]
> 
> 模板文件名：`post.md`
> 模板内容：
>
> ```
> ---
> title: {{ title }}
> date: {{ date }}
> updated: {{ date }}
> tags:
> categories:
> keywords:
> description:
> top_img:
> comments:
> cover:  # 文章缩略图
> toc:  # 是否显示目录
> toc_number:  # 是否显示目录编号
> copyright:  # 是否显示版权声明
> copyright_author:  # 作者
> copyright_author_href:  # 作者链接
> copyright_url:  # 文章链接
> copyright_info:  # 版权声明
> ---
> ```
> 

## 预览和部署

文章管理命令：

```bash
# 生成草稿预览
hexo server --draft

# 将草稿发布为文章
hexo publish draft "文章名"

# 清理缓存
hexo clean

# 生成静态文件
hexo generate

# 本地预览
hexo server
```


## 文章Formatter示例

```
---
title: 文章标题
date: 2024-01-20 14:30:00
updated: 2024-01-20 14:30:00
tags: 
  - 标签1
  - 标签2
categories: 
  - 分类1
  - 分类2
keywords: 关键词1,关键词2
description: 文章描述
top_img: https://example.com/image.jpg
comments: true
cover: https://example.com/cover.jpg
toc: true
toc_number: true
copyright: true
---
```

## 图片使用外链

`PicGo` 上传图片到 `GitHub` 图床，然后使用外链。

```
![图片描述](https://example.com/image.jpg)
```

## 工作流

我习惯使用草稿工作流。即

```
# 1. 创建草稿
hexo new draft "新文章"

# 2. 编辑草稿
# source/_drafts/新文章.md

# 3. 预览草稿
hexo server --draft

# 4. 发布草稿
hexo publish draft "新文章"
```

### 进阶使用

在`scaffolds/`下创建自定义目标文件。例如

`scaffolds/custom.md`

```
---
title: {{ title }}
date: {{ date }}
categories: 技术
tags:
  - 编程
  - 开发
cover: 
description:
---
```

```bash
# 创建多级目录的文章
hexo new post --path custom/2024/article-name "文章标题"


# 使用自定义模板创建特定路径的草稿
hexo new draft tech --path drafts/tech/article-name "文章标题"
```




