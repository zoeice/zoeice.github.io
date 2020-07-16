---
layout:     post
title:      "Glide加载jpg图片变绿的解决办法"
subtitle:   " \"目前看来比较靠谱的办法\""
description: 最近发现一个问题，用Glide加载图片，某几张jpg图的白色区域会变成淡绿色。找了很多解决办法，发现有的太影响性能，有的处理起来比较麻烦。最后在Github上找到了比较合适的解决方案，修改如下：
date:        2016-08-15 16:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg?x-oss-process=image/resize,w_380
catalog:    false
category:   android
tags:
    - Bug
    - Android
---


>最近发现一个问题，用Glide加载图片，某几张jpg图的白色区域会变成淡绿色。
找了很多解决办法，发现有的太影响性能，有的处理起来比较麻烦。
最后在Github上找到了比较合适的解决方案，修改如下：

方案1:  

```java
//设置缓存模式：缓存和imageView大小匹配的图
Glide.with(context)
      .load(url)
      .diskCacheStrategy(DiskCacheStrategy.SOURCE)
      .into(imageView);
```
方案2:

```java
//提高图片质量
Glide.with(context)
      .load(url)
      .asBitmap()
      .encoder(new BitmapEncoder(Bitmap.CompressFormat.PNG, 100))
      .into(imageView);
```
