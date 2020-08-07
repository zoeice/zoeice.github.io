---
layout:     post
title:      Mac上安装Java以及多版本切换
subtitle:   Java环境配置
description: Mac上打开终端，输入 java -version, 如果有显示Java版本信息，则系统已经有java环境，如果显示异常，则需要重新安装。
date:        2020-07-25 20:00:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-java.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-java.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   环境配置
tags:
    - 开发环境
    - Rust
---

## 检测环境
Mac上打开终端，输入`java -version`, 如果有显示Java版本信息，则系统已经有java环境，如果显示异常，则需要重新安装。
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-java-version.jpg)

## 安装
如果没有安装，可以去这里下载指定版本 [JDK官方下载](https://www.oracle.com/java/technologies/javase-downloads.html)
我们以java11为例子：
找到 **Java SE 11(LTS)**, 点击 **JDK Download**
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-java-11tls.jpg)

下载到 jdk-11.0.8_osx-x64_bin.dmg， 打开安装，一路Next
按键装好后继续用`java -version`看下环境有没有问题

## 配置环境
如果还是显示异常。
执行`open ~/.bash_profile`做环境配置，输入：
```shell
# Java
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-11.0.8.jdk/Contents/Home"
```
然后执行命令`source ~/.bash_profile`让配置生效。
再次执行`java -version`就发现版本显示正常了。

>这里如果是用的是zsh，也可以打开`open ~/.zshrc`来代替bash_profile配置

## 多版本Java切换
如果安装有多个java版本，怎么切换呢
执行`open ~/.bash_profile`做环境配置，输入
```shell
# Java
export JAVA_11_HOME="/Library/Java/JavaVirtualMachines/jdk-11.0.8.jdk/Contents/Home"
export JAVA_13_HOME="/Library/Java/JavaVirtualMachines/jdk-13.0.2.jdk/Contents/Home"
export JAVA_HOME=$JAVA_11_HOME
alias jdk11="export JAVA_HOME=$JAVA_11_HOME"
alias jdk13="export JAVA_HOME=$JAVA_13_HOME"
```
然后执行命令`source ~/.bash_profile`让配置生效。
接下来就可以执行命令`jdk11`或者`jdk13`来切换java版本了。
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-java-jdk11.jpg)
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-java-jdk13.jpg)