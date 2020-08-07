---
layout:     post
title:      React Hook学习(useState)
subtitle:   useState的使用与注意点
description: useState的使用与注意点
date:        2020-07-24 12:30:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-react.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-react.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   React
tags:
    - React
    - Hook
---

## 使用示例
以前在类组件中使用state是这样的：
```javascript
import React from 'react';

class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
        <div>
            <p>You clicked {this.state.count} times</p>
            <button onClick={() => this.setState({ count: this.state.count + 1 })}>
                Add
            </button>
        </div>
    );
  }
}
```

在函数组件里，没有this, 不能分配读取this.state, 有了Hook以后，就很简单了
```javascript
import React, { useState } from 'react';

function Example() {
	const [count, setCount] = useState(0);

	return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Add
            </button>
        </div>
	);
}

export default Example;
```

useState 唯一的参数就是初始值
useState 会返回一个数组：一个 state，一个更新 state 的函数
```javascript
// 这里可以任意命名，比如updateCount，因为返回的是数组，数组解构
const [count, setCount] = useState(initialValue);
```

>注意： setCount类似类组件的 this.setState，但是它不会把新的 state 和旧的 state 进行合并，而是直接替换


## 遇到的疑问：

### 1. setState是“异步”的，同步使用会有问题， 比如：
```javascript
import React, { useState } from 'react';

function Example() {
    const [num1, setNum1] = useState(0);
    const [num2, setNum2] = useState(0);
    const [result, setResult] = useState(0);

    function add(){
        setNum1(1)
        setNum2(2)
        setResult(num1 + num2)
    }

	return (
        <div>
            <p>num1 = {num1}</p>
            <p>num2 = {num2}</p>
            <p>result = {result}</p>
            <button onClick={() => add()}>Add</button>
        </div>
	);
}

export default Example
```

预想的结果是 `result = 3`, 但是运行后发现结果是这样的：
![](https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/content/post-hook-useState-result.jpg)

因为setNum1和setNum2都是“异步”的，add()函数执行时候，取得的num1和num2还是当前状态。
再次点击Add按钮后，结果就正确了。

可以把add()改一下：
```javascript
function add(){
    const n1 = 1, n2 = 2
    setNum1(n1)
    setNum2(n2)
    setResult(n1 + n2) 
}
```
这样就执行正常了。

### 2. state的值变化后的渲染
我们知道state里的值没发生变化，是不会执行组件函数了
```javascript
import React, { useState } from 'react';

function Example() {
    console.log('do render >>')
    const [num, setNum] = useState(0);

    const add = () => {
        setNum(1)
    }

	return (
        <div>
            <p>num = {num}</p>
            <button onClick={add}>Add</button>
        </div>
	);
}

export default Example
```

这个例子我们发现两个问题：

**问题1**：<br>
每次state变化一次，都会连续执行两次`do render >>`<br>
这是因为启动了 [**严格模式**](https://zh-hans.reactjs.org/docs/strict-mode.html)<br>
在严格模式下，React会执行两次render以检查额外的副作用。

>严格模式(StrictMode)仅在开发模式下运行；它们不会影响生产构建。

**问题2**：<br>
第一次点Add后，num从0变到1， 日志出现了 2次 `do render >>`<br>
第二次点Add后，num还是1没变化，日志还是出现了 2次 `do render >>`<br>
第三次点Add后，num还是1没变化，就没有日志了 <br>
问题持续关注：[useState not bailing out when state does not change](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffacebook%2Freact%2Fissues%2F14994)

### 3.state的同步和“异步”
```javascript
import React, { useState } from 'react';

function Example() {
    console.log('do render >>')
    const [num1, setNum1] = useState(0);
    const [num2, setNum2] = useState(0);

    const update = () => {
        console.log('update start')
        setNum1(1)
        console.log('update num1')
        setNum2(2)
        console.log('update num2')
    }

    const asyncUpdate = () => {
        console.log('-----------------------------')
        setTimeout(() => {
            console.log('async update start')
            setNum1(4)
            console.log('async update num1')
            setNum2(5)
            console.log('async update num2')  
        })
    }

	return (
        <div>
            <p>num1 = {num1}</p>
            <p>num2 = {num2}</p>
            <button onClick={update}>update</button>
            <button onClick={asyncUpdate}>asyncUpdate</button>
        </div>
	);
}

export default Example
```

依次点 **update** 和 **asyncUpdate**， 我们发现日志：
```shell
update start
update num1
update num2
do render >>
do render >>
-----------------------------
async update start
do render >>
do render >>
async update num1
do render >>
do render >>
async update num2
```

根本原因是setState并不是真正意义上的异步操作，它只是模拟了异步的行为。React中会去维护一个标识（isBatchingUpdates），判断是直接更新还是先暂存state进队列。

只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。

### 4.如何拿到useState更新后的值
原生的state可以用过setState(partialState, callback)里的callback及时拿到更新后的值，但是在useState里怎么拿到呢？
可以使用useEffect模拟

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        if (count > 1) {
            console.log('over 1 call');
        } else {
            console.log('No call');
        }
    }, [count]);

	return (
        <div>
            <p>count = {count}</p>
            <button type="button" onClick={() => setCount(count + 1)}>
                Add
            </button>
        </div>
	);
}

export default Example
```

看起来逻辑散落在两个地方，我们使用自定义Hook修改下让它看起来更内聚：
```javascript
import React, { useState, useEffect } from 'react';

const useStateWithCallback = (initialState, callback) => {
    const [state, setState] = useState(initialState);
  
    useEffect(() => callback(state), [state, callback]);
  
    return [state, setState];
  };

function Example() {
    const [count, setCount] = useStateWithCallback(0, count => {
        if (count > 1) {
            console.log('over 1 call');
        } else {
            console.log('No call');
        }
    });

	return (
	    <div>
            <p>count = {count}</p>
            <button type="button" onClick={() => setCount(count + 1)}>
                Add
            </button>
	    </div>
	);
}

export default Example
```
这样看起来好多了，可以把 `useStateWithCallback` 抽离到独立文件，整个项目都可以调用。