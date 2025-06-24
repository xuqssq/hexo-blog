---
title: Logging用法
tags:
  - Python
  - log
  - logging
categories:
  - Python
  - logging
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

# Logging用法

## 打印所有Logger对象

```python
 for name in logging.Logger.manager.loggerDict.keys():
        logger = logging.getLogger(name)
        print('name = %s, logger = %s' % (name, logger))	
```

