---
layout:     post
title:      "iOS中编程思想"
subtitle:   ""
date:       2016-08-23 15:37:00
author:     "ZoeIce"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - iOS
    - Swift
    - Objective-C
    - 编程思想
---

## 编程思想分类   
1.面向过程    
以过程为中心的编程思想，分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了。

2.面向对象    
以事物为中心的编程思想，特性有：对象唯一性，抽象性，继承性，多态性。

3.链式编程思想    
是将多个操作（多行代码）通过点号(.)链接在一起成为一句代码,使代码可读性好。a(1).b(2).c(3)

>特点：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值）   

Objective-C:

```objc
@interface MyTest()

@property (copy, nonatomic) void (^actionBlock)(NSString *param);

@end

@implementation MyTest

- (void)funPublic {
    self.fun1(@"1").fun2(@"2");
}

- (MyTest * (^)(NSString *param))fun1 {
    return ^(NSString *param) {
        return self;
    };
}

- (MyTest * (^)(NSString *param))fun2 {
    return ^(NSString *param) {
        return self;
    };
}

@end
```

Swift:  

```swift
class MyTest : NSObject {

    func funPublic() {
        fun1("1").fun2("2")
    }

    var fun1 : (String) -> MyTest {
        return {
            print($0)
            return self
        }
    }

    var fun2 : (String) -> MyTest {
        return {
            print($0)
            return self
        }
    }
}
```

4.响应式编程思想   
响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播

5.函数式编程思想   
是把操作尽量写成一系列嵌套的函数或者方法调用。特性：高阶函数，偏应用函数，柯里化，闭包。

>特点：每个方法必须有返回值（本身对象）,把函数或者Block当做参数,block参数（需要操作的值）block返回值（操作结果）

6.函数响应式编程（FRP）
