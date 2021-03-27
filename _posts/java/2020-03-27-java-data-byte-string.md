---
layout:     post
title:      Java里byte[]和String转换不一致的坑
subtitle:   解决byte[]转成String再转成byte[]结果与期望不符合的问题
description: 最近做Java项目时对数据做格式转换，发现踩了一个坑。
date:        2020-03-27 20:00:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-java.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-java.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   环境配置
tags:
    - 问题解决
    - Java
---

## 背景
最近做Java项目时对数据做格式转换，发现踩了一个坑。

看下问题场景, 把`byte[]转换成String，再转回byte[]`，发现结果和原来不一样：
~~~java
byte[] a = {-76};
System.out.println(Arrays.toString(a));
String aStr = new String(a);
byte[] b = aStr.getBytes();
System.out.println(Arrays.toString(b));
~~~
打印出来：
~~~bash
[-76]
[-17, -65, -67]
~~~

竟然不一样，跟期望的完全不一样 -_-!~

看了下 String.getBytes() 带传参的方法是：
~~~java
public byte[] getBytes(@RecentlyNonNull Charset charset) {
    throw new RuntimeException("Stub!");
}
~~~
咱打印下Charset默认的编码方式：
~~~bash
System.out.println(Charset.defaultCharset().name());
~~~
打印结果是
~~~bash
UTF-8
~~~

查了下资料，UTF-8是一种[多位元组序列](https://zh.wikipedia.org/zh/UTF-8)，需要用多字节来表示一个字符。

编码成字符串后，再转回byte[]就会和原来不一致。

制定一个单字节编码方式试一试
~~~java
byte[] a = {-76};
System.out.println(Arrays.toString(a));
String aStr = new String(a, Charset.forName("ISO-8859-1"));
byte[] b = aStr.getBytes(Charset.forName("ISO-8859-1"));
System.out.println(Arrays.toString(b));
~~~
打印结果是
~~~bash
[-76]
[-76]
~~~

符合期望~