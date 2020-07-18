---
layout:     post
title:      "iOS中的链式编程"
subtitle:   ""
description: 1.面向过程 以过程为中心的编程思想，分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。
date:        2016-08-23 15:37:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-chain.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-chain.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   iOS
tags:
    - iOS
    - Swift
    - Objective-C
---

## 链式编程   
提到链式编程，在Objective-C中最典型的代表就是Masonry了，比较完美的实现了函数式和链式编程。
链式编程一般就是将多个操作（多行代码）通过点号(.)链接在一起成为一句代码,使代码可读性好。a(1).b(2).c(3)

>特点：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值）   

## 例子
我们想要调用一个类对象中的两个方法，正常的实现方式是：
Objective-C:

NormalClass.h
```objc
#import <Foundation/Foundation.h>

@interface NormalClass : NSObject

- (void)fun1: (NSString *)param;
- (void)fun2: (NSString *)param;

@end
```

NormalClass.m
```objc
#import "NormalClass.h"

@implementation NormalClass

- (void)fun1: (NSString *)param {
    NSLog(@"normal fun1: %@", param);
}

- (void)fun2: (NSString *)param {
    NSLog(@"normal fun2: %@", param);
}

@end
```

调用如下：
```objc
NormalClass *normal = [NormalClass new];
[normal fun1: @"param1"];
[normal fun2: @"param2"];
```

正常输出：
```
normal fun1: param1
normal fun2: param2
```

如果要用链式来实现呢，可以这样写：

ChainClass.h
```objc
#import <Foundation/Foundation.h>

@interface ChainClass : NSObject

@property (copy, nonatomic) void (^actionBlock)(NSString *param);

- (ChainClass * (^)(NSString *param))fun1;
- (ChainClass * (^)(NSString *param))fun2;
    
@end
```

ChainClass.m
```objc
#import "ChainClass.h"

@implementation ChainClass

- (ChainClass * (^)(NSString *param))fun1 {
    return ^(NSString *param) {
        NSLog(@"chain fun1: %@", param);
        return self;
    };
}

- (ChainClass * (^)(NSString *param))fun2 {
    return ^(NSString *param) {
        NSLog(@"chain fun2: %@", param);
        return self;
    };
}

@end
```
调用如下：
```objc
ChainClass *chain = [ChainClass new];
chain.fun1(@"param1").fun2(@"param2");
```

正常输出：
```
chain fun1: param1
chain fun2: param2
```


链式调用在Swift里实现起来就简单很多， 如下
Swift:  

```swift
import Foundation

class ChainClass: NSObject {
    var fun1 : (String) -> ChainClass {
        return {
            NSLog("chain fun1: %@", $0)
            return self
        }
    }

    var fun2 : (String) -> ChainClass {
        return {
            NSLog("chain fun2: %@", $0)
            return self
        }
    }
}
```

调用如下：
```swift
ChainClass().fun1("param1").fun2("param2")
```

正常输出：
```
chain fun1: param1
chain fun2: param2
```
