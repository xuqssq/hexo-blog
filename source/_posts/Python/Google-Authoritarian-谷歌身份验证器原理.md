---
title: Google Authoritarian/谷歌身份验证器原理
tags:
  - 爬虫
  - Mac验证算法
categories:
  - Python
toc: true
toc_number: true
abbrlink: 2984
date: 2022-11-06 09:26:57
updated:
keywords:
description:
top_img: https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/night-min.jpg
comments:
cover: https://cdn.jsdelivr.net/gh/daojiAnime/cdn@master/img/night-min.jpg
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

> TOTP算法(Time-based One-time Password algorithm)是一种从共享密钥和当前时间计算一次性密码的算法。 它已被采纳为Internet工程任务组标准RFC 6238，是Initiative for Open Authentication（OATH）的基石，并被用于许多双因素身份验证系统。
>
> TOTP是基于散列的[消息认证码](https://baike.baidu.com/item/消息认证码/1354818?fromModule=lemma_inlink)（HMAC）的示例。 它使用加密哈希函数将密钥与当前时间戳组合在一起以生成一次性密码。 由于网络延迟和不同步时钟可能导致密码接收者必须尝试一系列可能的时间来进行身份验证，因此时间戳通常以30秒的间隔增加，从而减少了潜在的搜索空间。

### TOTP算法使用场景

&emsp;&emsp;TOTP算法的使用场景可以有动态口令认证、前后端接口认证等，TOTP算法需要客户端和服务端保持时钟一致(基于UTC时间)

## 适用场景

- 服务器登录动态密码验证
- 公司VPN登录双因素验证
- 银行转账动态密码
- 网银、网络游戏的实体动态口令牌
- 等基于时间有效性验证的应用场景

## TOTP的基本原理

### TOTP计算公式

```python
TOTP(K, TC) = Truncate(HMAC-SHA-1(K, TC))
```

K，密钥串 HMAC-SHA-1， 表示使用SHA-1做HMAC（当然也可以使用SHA-256等） C，基于时间戳计算得出，通过定义纪元（T0）的开始并以时间间隔（TI）为单位计数，将当前时间戳变为整数时间计数器（TC） Truncate，是一个函数，用于截取加密后的字符串

### TC的计算公式

```python
TC = (T - T0) / T1;
```

T，当前的时间戳 T0，起始时间，一般为0 T1，时间间隔，根据业务需要自定义

### Truncate函数

1. 取加密后的最后一个字节的的低4位，offset；
2. 以offset开始取4个字节，按照大端方式组成整数，binary；
3. 根据需要的长度对binary取模，opt
4. 以字符串方式返回opt，并补足长度

```python
h = hmac.new(self.key.encode(), msg, sha256).digest()
offset = h[len(h)-1] & 0xf
binary = (h[offset] & 0x7f) << 24
binary = binary | ((h[offset+1] & 0xff)<<16)
binary = binary | ((h[offset+2] & 0xff)<<8)
binary = binary | (h[offset+3] & 0xff)
otp = binary % (10 ** self.codeDigits)
return str(otp).rjust(self.codeDigits, '0')
```

## Python实现

```python
import binascii
import hmac
import time
from hashlib import sha256

class TOTP:
    
    def __init__(self, key, codeDigits):
        self.key = key
        self.codeDigits = codeDigits

    def truncate(self, time):
        time = time.rjust(16,'0')
        bigint = binascii.unhexlify(hex(int('10'+time, 16))[2:])
        msg = bigint[1:len(bigint)]
        h = hmac.new(self.key, msg, sha256).digest()
        offset = h[len(h)-1] & 0xf
        binary = (h[offset] & 0x7f) << 24
        binary = binary | ((h[offset+1] & 0xff)<<16)
        binary = binary | ((h[offset+2] & 0xff)<<8)
        binary = binary | (h[offset+3] & 0xff)
        otp = binary % (10 ** self.codeDigits)
        return str(otp).rjust(self.codeDigits, '0')

    def tc(self, ttl):
        return format(int(int(time.time())/int(ttl)),'x').upper()
```

上面的代码就是我基于python3的实现（可以保存为totp.py），散列算法使用的是SHA-256，使用方式如下：

```python
import totp
import base64

secretKey = base64.b32encode(b'My secret key')
t = totp.TOTP(secretKey, 4)
time = t.tc(60)    # 此处时间单位为秒
result=t.truncate(time)
print(result)
```
