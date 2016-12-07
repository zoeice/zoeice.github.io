---
layout:     post
title:      "CocoaPods安装和版本升级"
subtitle:   " \"配置Pod和遇到的问题解决\""
date:       2016-08-18 11:38:00
author:     "ZoeIce"
header-img: "img/post-bg-cocoapods.jpg"
catalog: true
tags:
    - CocoaPods
    - iOS
---

## 配置ruby源
翻墙用户直接跳过配置ruby源的步骤。[开始进行 pod的下载和安装](#build)  

因为`cocoapods.org`被墙了，解决办法是，我们可以用阿里云的Ruby镜像来访问cocoapods。按照下面的顺序在终端中敲入依次敲入命令：  

首先，检查你的ruby源：  

```Shell
$ gem sources -l
```
默认情况下，终端应该返回如下信息：

```Shell
*** CURRENT SOURCES ***

https://rubygems.org/
```

当然这个源在墙内是访问不到的。因此我们需要寻找一个可以在国内访问到的镜像。目前笔者找到的是`http://rubygems-china.oss.aliyuncs.com`这个阿里云的镜像，当然随着时间的推移，未来这个镜像也有可能无法访问了，到时候就只能重新寻找了。

　　确认镜像可用后，现在就要开始修改ruby源了。首先执行以下命令删除原来的ruby源：

```Shell
$ gem sources --remove https://rubygems.org/
```
执行命令后可在终端看见以下信息：   

```Shell
https://rubygems.org/ removed from sources
```

然后下一步添加你找到的可用的镜像源：

```Shell
$ gem sources -a http://rubygems-china.oss.aliyuncs.com
```
此时如果你再执行gem sources -l命令，就能看到当前镜像源里只有阿里云这一个了。

<p id = "build"></p>
---

## pod的下载和安装   

mac系统已经默认安装好Ruby环境，现在可以直接下载和安装CocoaPods。 打开终端输入

```Shell
$ sudo gem install cocoapods
```
安装好以后，可以查看pod版本，确认是否安装成功

```Shell
$ pod --version
```
如果在终端看到如下类似的版本号

```Shell
1.1.1
```
表示已经安装成功。 此时pod还不能使用，需要下载pod所有源的索引文件。在终端输入如下命令：

```Shell
$ pod setup
```
这个过程很慢。

## 遇到的问题

如果过程中出现红色的错误提醒。按如下几个可能的原因排查问题：

#### 1. gem版本太低   

将gem升级到最新版本，在终端输入：   

```Shell
$ sudo gem update --system
```

#### 2. 连不上github

完成后如果还不行，在终端输入`ping github.com`是否可以ping成功

#### 3. .cocoapods目录下的配置信息有问题

如果还不行，在终端输入：

```Shell
$ pod repo list
```

如果显示`0 repos`说明没有安装成功。
在终端输入：

```Shell
$ cd ~/.cocoapods
```

进入cocoapods文件后在终端输入:

```Shell
$ du -sh *
```

重新执行`pod setup`，过一段时间后提示`setup completed`,在终端中输入 `pod repo list`，展示出安装列表；

```Shell
artsy
- Type: git (master)
- URL:  https://github.com/Artsy/Specs.git
- Path: /Users/apple/.cocoapods/repos/artsy

master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/apple/.cocoapods/repos/master

2 repos
```

这样总算安装好了。接下来输入：

```Shell
$ pod search AFNetworking
```

输入过后它可能会报
`[!] Unable to find a pod with name, author, summary, or descriptionmatching `AFNetworking`
解决方案是输入：

```Shell
rm ~/Library/Caches/CocoaPods/search_index.json
```

后在一次输入：`pod search AFNetworking`

就可以咯。
