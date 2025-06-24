---
title: 线程同步锁Synchronized
tags:
  - Java
toc: false
abbrlink: 9083
date: 2018-08-22 16:54:59
categories:
description:
---
![](https://ws1.sinaimg.cn/large/e3bf8736gy1fyqgc51a2lj22401eo156.jpg)
<!--more-->

# 线程同步锁Synchronized

```java

class AccountingSync implements Runnable {
    static AccountingSync instance = new AccountingSync();
    static int i = 0;

    @Override
    public void run() {
        //省略其他耗时操作....
        while (true) {
            synchronized (AccountingSync.class) {
                if (i < 100) {
                    i++;
                    System.out.println(Thread.currentThread().getName() + "    "+ i);
                }
            }
        }
    }

}

class test {
    public static void main(String[] args) {
        AccountingSync accountingSync = new AccountingSync();
        Thread t1 = new Thread(accountingSync);
        Thread t2 = new Thread(accountingSync);
        t1.start();
        t2.start();
    }
}
```