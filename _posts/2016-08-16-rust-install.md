---
layout:     post
title:      "在Linux和Mac上安装Rust"
subtitle:   " \"配置Rust开发环境\""
description: 如果使用 Linux 或 Mac，所有我们需要做的就是打开一个终端并输入如下
date:        2016-08-16 11:38:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-ruby.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-ruby.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   环境配置
tags:
    - 开发环境
    - Rust
---

## 安装Rust  
如果使用 Linux 或 Mac，所有我们需要做的就是打开一个终端并输入如下：  

```Shell
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

这将会下载一个脚本，并开始安装。如果一切顺利，你将会看到这些：  

```Shell
Rust is ready to roll.
```

在这里输入，输入`y`来选择`yes`，并按照接下来的提示操作。

## 卸载  
卸载 Rust 跟安装它一样容易。在 Linux 或 Mac 上，运行卸载脚本：

```Shell
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

## 查看版本号  
安装了 Rust 后，我们可以打开一个 shell，并输入：

```Shell
$ rustc --version
```

你应该看到版本号，提交的 hash 值和提交时间。  

```Shell
rustc 1.10.0 (cfcb716cf 2016-07-03)
```

如果你做到了，那么 Rust 已成功安装！恭喜你
