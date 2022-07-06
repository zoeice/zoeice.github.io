---
layout:     post
title:      Javascript对象的遍历方法
subtitle:   7种遍历方法对比
description: 
date:        2020-08-23 10:41:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-js.gif
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-es.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   Javascript
tags:
    - Javascript
---

## 前言：
我们先定义一个完整对象， 后面的几种遍历方式都基于这个对象来处理，方便对比：
```javascript
const obj = Object.create(
    {
        proto_enum: 'proto-enum-value',
        [Symbol('proto_enum_s')]: 'proto-enum-symbol-value'
    }, 
    {
        unenum:  {
            value: 'unenum-value',
            enumerable: false
        },
        [Symbol('unenum_s')]: {
            value: 'unenum-symbol-value',
            enumerable: false
        }
    }
)
obj.enum = 'enum-value'
obj[Symbol('enum_s')] = 'enum-symbol-value'
```

看控制台比较直观：

![image-20220623135321572](http://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-js-traverse-object.jpg)

其中半透明部分就是不可枚举

Prototype里的就是原型链



对比下参数的类型

| key          | 原型属性 | 可枚举 | Symbol属性 | 值                        |
| ------------ | -------- | ------ | ---------- | ------------------------- |
| proto_enum   | ✔️        | ✔️      |            | 'proto-enum-value'        |
| proto_enum_s | ✔️        | ✔️      | ✔️          | 'proto-enum-symbol-value' |
| enum         |          | ✔️      |            | 'enum-value'              |
| enum_s       |          | ✔️      | ✔️          | 'enum-symbol-value'       |
| unenum       |          |        |            | 'unenum-value'            |
| unenum_s     |          |        | ✔️          | 'unenum-symbol-value'     |



## 1. For in

`for in` 循环是最基础的遍历对象的方式，它还会得到对象原型链上的属性。
>Tips: 它不会返回对象的不可枚举属性。

```javascript
for (let key in obj) {
    console.log(obj[key]) 
}
```
运行结果：
```bash
enum-value
proto-enum-value
```
发现原型链上的属性和自身的属性都被打印出来了， 不可枚举属性被过滤掉了



如果要过滤掉原型链的属性，可以这样处理：

```javascript
for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
        console.log(obj[key]) 
    }
}
```
运行结果：
```bash
enum-value
```

看出来原型链上的属性被过滤掉了

## 2. Object.keys()

`Object.keys()` 是 ES5 的一个对象方法，该方法返回对象自身属性名组成的数组，它会自动过滤掉原型链上的属性
>Tips: 它不会返回对象的不可枚举属性。

```javascript
Object.keys(obj).forEach((key) => {
    console.log(obj[key])
})
```

运行结果：

```bash
enum-value
```

可以看出它只会返回自身属性，并且会过滤掉不可枚举属性

## 3. Object.values()

如果想直接遍历值，可以用`Object.values()`

```javascript
Object.values(obj).forEach((v) => {
    console.log(v)
})
```
运行结果：
```bash
enum-value
```

可以看出它只会返回自身属性，并且会过滤掉不可枚举属性

## 4. Object.entries()

想同时拿到key和值，也可以用`Object.entries()`

```javascript
Object.entries(obj).forEach((item) => {
    console.log(item[0], item[1]) 
})
```
运行结果：
```bash
enum enum-value
```

可以看出它只会返回自身属性，并且会过滤掉不可枚举属性

## 5. Object.getOwnPropertyNames()

`Object.getOwnPropertyNames()` 也是 ES5 的一个对象方法，该方法返回对象自身属性名组成的数组。
>Tips: 它会返回对象的不可枚举属性。

```javascript
Object.getOwnPropertyNames(obj).forEach((key) => {
    console.log(obj[key]) 
})
```
运行结果：
```bash
unenum-value
enum-value
```

可以看出它只会返回自身属性，包括不可枚举属性

## 6. Object.getOwnPropertySymbols()

ES2015 新增了 Symbol 数据类型，该类型可以作为对象的键，针对该类型 ES2015 同样新增`Object.getOwnPropertySymbols() `方法

```javascript
Object.getOwnPropertySymbols(obj).forEach((key) => {
		console.log(obj[key])
})
```

运行结果：

```javascript
unenum-symbol-value
enum-symbol-value
```

可以看出它只能拿到symbol类型的属性，包括不可枚举类型



## 7. Reflect.ownKeys()

Reflect.ownKeys() 方法是 ES2015 的静态方法，该方法返回对象自身所有属性名组成的数组
>Tips: 它会返回对象的不可枚举属性。
```javascript
Reflect.ownKeys(obj).forEach((key) => {
    console.log(obj[key])
})
```
运行结果：
```bash
unenum-value
enum-value
unenum-symbol-value
enum-symbol-value
```

可以看出它会返回所有的自身属性，包括Symbol类型



对比一下这几种方式的输出结果

|                                | 含自身属性 | 含原型属性 | 含不可枚举 | 含Symbol属性 |
| ------------------------------ | ---------- | ---------- | ---------- | ------------ |
| for in                         | ✔️          | ✔️          |            |              |
| Object.keys()                  | ✔️          |            |            |              |
| Object.values()                | ✔️          |            |            |              |
| Object.entries()               | ✔️          |            |            |              |
| Object.getOwnPropertyNames()   | ✔️          |            | ✔️          |              |
| Object.getOwnPropertySymbols() |            |            | ✔️          | ✔️            |
| Reflect.ownKeys()              | ✔️          |            | ✔️          | ✔️            |

