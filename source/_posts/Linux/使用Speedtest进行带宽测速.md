---
title: 使用Speedtest进行带宽测速
tags:
  - Linux
toc: false
abbrlink: 18198
date: 2018-04-15 18:33:48
categories:
description:
---
# 使用Speedtest进行带宽测速 #
--------------------------------------
```shell
wget -O speedtest-cli https://raw.github.com/sivel/speedtest-cli/master/speedtest.py
chmod +x speedtest-cli
```

---------------------------------------
```shell
wget https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py

chmod a+rx speedtest.py
mv speedtest.py /usr/local/bin/speedtest-cli
chown root:root /usr/local/bin/speedtest-cli

speedtest-cli
```
