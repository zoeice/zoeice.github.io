---
layout:     post
title:      Javascript对象的遍历方法
subtitle:   5种遍历方法对比
description: 
date:        2022-06-23 10:41:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-js.gif
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-es.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   Javascript
tags:
    - Javascript
---

## 1. For in
`for in` 循环是最基础的遍历对象的方式，它还会得到对象原型链上的属性。
>Tips: 它不会返回对象的不可枚举属性。

```javascript
// 创建一个对象并指定其原型：
const obj = Object.create({
 proto: 'a'
})
 
// 为对象增加2个自身的属性
obj.own1 = 'b'
obj.own2 = 'c'

for (let key in obj) {
    console.log(obj[key]) 
}
```
运行结果：
```
[LOG]: "a" 
[LOG]: "b" 
[LOG]: "c" 
```
发现原型链上的属性和自身的属性都被打印出来了

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
[LOG]: "b" 
[LOG]: "c" 
```

## 2. Object.keys()
`Object.keys()` 是 ES5 的一个对象方法，该方法返回对象自身属性名组成的数组，它会自动过滤掉原型链上的属性
>Tips: 它不会返回对象的不可枚举属性。

```javascript
const obj = Object.create({
 proto: 'a'
})
obj.own1 = 'b'
obj.own2 = 'c'

Object.keys(obj).forEach((key) => {
    console.log(obj[key])
})
```

如果想直接遍历值，可以用`Object.values()`
```javascript
Object.values(obj).forEach((v) => {
    console.log(v)
})
```
运行结果：
```
[LOG]: "b" 
[LOG]: "c" 
```

想同时拿到key和值，也可以用`Object.entries()`
```javascript
Object.entries(obj).forEach((item) => {
    console.log(item[0], item[1]) 
})
```
运行结果：
```
[LOG]: "own1",  "b" 
[LOG]: "own2",  "c" 
```

## 3. Object.getOwnPropertyNames()
`Object.getOwnPropertyNames()` 也是 ES5 的一个对象方法，该方法返回对象自身属性名组成的数组。
>Tips: 它会返回对象的不可枚举属性。

```javascript
const obj = Object.create({
    proto: 'a'
}, {
    own_unenum: {
        value: 'unenum',
        enumerable: false
    }
})
obj.own1 = 'b'
obj.own2 = 'c'

// 不包括不可枚举的属性
Object.keys(obj).forEach((key) => {
    console.log('keys() >>', obj[key]) 
})
 
// 包括不可枚举的属性
Object.getOwnPropertyNames(obj).forEach((key) => {
    console.log('getOwnPropertyNames() >>', obj[key]) 
})
```
运行结果：
```
[LOG]: "keys() >>",  "b" 
[LOG]: "keys() >>",  "c" 
[LOG]: "getOwnPropertyNames() >>",  "unenum" 
[LOG]: "getOwnPropertyNames() >>",  "b" 
[LOG]: "getOwnPropertyNames() >>",  "c" 
```

## 4. Reflect.ownKeys()
Reflect.ownKeys() 方法是 ES2015 的静态方法，该方法返回对象自身所有属性名组成的数组
>Tips: 它会返回对象的不可枚举属性。
```javascript
const obj = Object.create({
    proto: 'a'
}, {
    own_unenum: {
        value: 'unenum',
        enumerable: false
    }
})
obj.own1 = 'b'
obj.own2 = 'c'

Reflect.ownKeys(obj).forEach((key) => {
    console.log(obj[key])
})
```
运行结果：
```
[LOG]: "unenum" 
[LOG]: "b" 
[LOG]: "c" 
```
