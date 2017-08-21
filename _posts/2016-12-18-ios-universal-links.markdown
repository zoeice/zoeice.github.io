---
layout:     post
title:      "iOS Universal Links支持"
subtitle:   "突破微信限制，直接跳转app或者网页"
date:       2016-12-18 15:37:00
author:     "ZoeIce"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - iOS
    - Objective-C
---

## 简介
当你支持 Universal Links以后，在 **iOS 9 及以上**，你点击链接( 例如 https://xxx.com/open?param=abc )可以不经过 Safari 直接跳转到安装的app里。

- 在 **Safari**，**微信**，**备忘录** 等所有地方，点击链接，就直接打开安装的app了。 你可以根据url带的参数param，自由决定要去app中的哪个页面。
- 当你没有安装该app的时候,你可以通过 Safari 打开 https://xxx.com/open?param=abc 。

对用户来说，这个过程非常自然，方便。

## Universal Links 优点
> 官方原文  
- **Unique.** Unlike custom URL schemes, universal links can’t be claimed by other apps, because they use standard HTTP or HTTPS links to your website.  
- **Secure.** When users install your app, iOS checks a file that you’ve uploaded to your web server to make sure that your website allows your app to open URLs on its behalf. Only you can create and upload this file, so the association of your website with your app is secure.  
- **Flexible.** Universal links work even when your app is not installed. When your app isn’t installed, tapping a link to your website opens the content in Safari, as users expect.  
- **Simple.** One URL works for both your website and your app.  
- **Private.** Other apps can communicate with your app without needing to know whether your app is installed.  

- **唯一性:** 不像自定义的scheme,因为它使用标准的http/https链接到你的web站点,所以它不会被其它的app所声明.另外,Custom URL scheme 因为是自定义的协议，所以在没有安装 app 的情况下是无法直接打开的，而 universal links 本身是一个 HTTP/HTTPS 链接，所以有更好的兼容性
- **安全:** 当用户的手机上安装了你的app,那么iOS将去你的网站上去下载你上传上去的说明文件(这个说明文件声明了你的app可以打开哪些类型的http链接).因为只有你自己才能上传文件到你网站的根目录,所以你的网站和你的app之间的关联是安全的.
- **可变:** 当用户手机上没有安装你的app的时候,Universal Links也能够工作.如果你愿意,在没有安装你的app的时候,用户点击链接,会在safari中展示你网站的内容.
- **简单:** 一个URL链接,可以同时作用于网站和app
- **私有:** 其它app可以在不需要知道你的app是否安装了的情况下和你的app相互通信.

添加 **universal links** 的支持很简单，有三个步骤：

## 1. 创建并上传关联文件  
- 创建一个json格式的关联文件，命令为 **apple-app-site-association**  
`注意，文件名必须是这个，不能加.json后缀`  

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            },
            {
                "appID": "ABCD1234.com.apple.wwdc",
                "paths": [ "*" ]
            }
        ]
    }
}
```  
说明:  
**appID:** 这里的组成方式是 {teamId}.{yourapp’s bundle identifier}。
如上面的例子，teamId 就是 9JA89QQLNQ， bundle identifier 就是 com.apple.wwdc。  
**paths:** 这里设定你的app支持的路径列表，只有匹配上这些路径的链接，才能被 Universal Links 识别处理。    
举个例子: 如果你的网站是xxx.com，你的path写的是 "/support/\*"，那么当用户点击 https://xxx.com/support/testPage ，就可以进入你的app了，相反 https://xxx.com/other 就不会。  
`path是大小写敏感的`  
*号表示任意路径.  
?号表示任意字符  
**details** 这里支持配置多个  

- 上传 **apple-app-site-association** 到你的Https服务器上。  
`注意：你必须要把这个文件放在你服务器的root根目录，或者 .well-known的子目录。`   
如下：  
`https://xxx.com/apple-app-site-association`  
或者  
`https://xxx.com/.well-known/apple-app-site-association`

## 2. 让你的app支持 Universal Links  
