---
layout:     post
title:      ES7-装饰器Decorator详解
subtitle:   Decorator的使用分析
description: 
date:        2020-07-26 08:30:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-js.gif
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-es.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   ES
tags:
    - ES
---

## Javascript中的装饰器
js中的装饰器是在ES7里的，它依旧依赖ES5里的 `Object.defineProperty` 方法。

先看下装饰器在代码中的样子：
~~~js
@frozen 
class Foo {
  @configurable(false)
  method() {}
}
~~~
@frozen 和 @configurable 就是我们说的装饰器。<br>
可以看出是通过`@`来使用装饰器。一共用了两种：一个用在类上，一个用在方法上。


## 讲讲Object.defineProperty
看下ES6里类的写法：
~~~js
class Cat {
    meow() {
        return `${this.name} says Meow!`;
    }
}
~~~

这个写法只是一个语法糖，这段代码实际上在执行时是这样的：
~~~js
function Cat() {}
Object.defineProperty(Cat.prototype, "meow", {
    value: function() { return `${this.name} says Meow!`; },
    enumerable: false,
    configurable: true,
    writable: true
});
~~~

`Object.defineProperty(target, name, descriptor)` 可以直接在一个对象上定义一个新属性，或者修改一个现有的属性，并返回这个对象。
defineProperty接受三个参数：
- `target`：要处理的对象
- `name`：要定义或修改的属性名称
- `descriptor`：将被定义或修改的属性描述符

## 属性描述符
定义的对象里，每一种属性都对应一个属性描述符对象，用来描述该属性的特性。<br>
目前有两种属性描述符：
- **数据描述符**：一个具有值的属性。该值会设置是否可被重写。
- **存取描述符**：由getter-setter函数对描述的属性
>属性描述符必须是这两种形式之一, 两种形式不共存。

数据描述符独有：
- **value**：改属性的值
- **writable**：为true表示该属性的value可以被重写

存取描述符独有的：
- **get**：获取该属性的访问器函数(getter)，当访问该属性时，该方法会被执行，方法执行时没有参数传入；如果没有 `getter` 则为 `undefined`
- **set**：该属性的设置器函数(setter)，当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值；如果没有则为 `undefined`，为该属性赋不了值

二者同时拥有的:
- **configurable**：为ture时，表示当前属性的‘属性表述符’对象可以被更改，该属性可以使用delete删除，否则就是不允许更改
- **enumerable**：为true时，表示当前属性可以被枚举，也就是当前属性是否可以在 `for...in` 循环和 `Object.keys()` 中被遍历出来

>一个没有get/set/value/writable定义的属性被称为"通用的"，并被认为是一个数据描述符

## 类的装饰
装饰器可以用来装饰整个类。
~~~js
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

// 显示true
console.log(MyTestableClass.isTestable)
~~~
上面代码中，`@testable` 就是一个装饰器。它修改了 `MyTestableClass` 这个类的行为，为它加上了静态属性 `isTestable`。`testable` 函数的参数 `target`是 `MyTestableClass` 类本身。


## 方法的装饰
装饰器不仅可以装饰类，还可以装饰类的属性。
比如我们把 `meow()` 设置成只读，如果使用装饰器的话可以这样写：
~~~js
function readonly(target, name, descriptor) {
    discriptor.writable = false;
    return discriptor;
}

class Cat {
    @readonly
    meow() {
        return `${this.name} says Meow!`;
    }
}
~~~

我们看到，定义装饰器的时候，参数有三个: `target`、`name`、`descriptor`，是不是和 `Object.defineProperty` 的入参很像。
其实，装饰器在作用于方法的时候，实际上是通过 `Object.defineProperty` 来进行扩展和封装的。
所以上面的这段代码中，装饰器实际是这样的：
~~~js
let descriptor = {
    value: function() { return `${this.name} says Meow!`; },
    enumerable: false,
    configurable: true,
    writable: true
};

descriptor = readonly(Cat.prototype, "meow", descriptor) || descriptor;

Object.defineProperty(Cat.prototype, "meow", descriptor);
~~~
这样就看得比较清楚了。

>注意点：
- 如果装饰器作用在类本身，我们操作的对象也是类本身。
- 如果装饰器作用在类的某个具体方法时，我们操作的对象是它的描述符（descriptor）。

