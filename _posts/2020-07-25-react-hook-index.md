---
layout:     post
title:      React Hook 学习（索引）
subtitle:   Hook是什么？有哪些？
description: Hook是React 16.8的新特性，它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性
date:        2020-07-24 11:56:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-react.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-react.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   React
tags:
    - React
    - Hook
---

## 介绍
Hook是React 16.8的新特性，它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

基础的Hook：
1. useState
2. useEffect
3. useContext

额外的Hook：
1. useReducer
2. useCallback
3. useMemo
4. useRef
5. useImperativeHandle
6. useLayoutEffect
7. useDebugValue

自定义Hook

## Hook能解决什么问题？
React 16 已经是 Javascript 圈子里非常受欢迎，且独特的框架，有这些优点：
- 支持组件的自由组合。可以构建简单的页面，也能构建大型的前端应用。
- 单项数据流的设计
- 高效的性能。基于虚拟DOM实现，可以实现基于数据变化才重新渲染。
- 分离的设计。分离为 ReactDOM 和 React.js，可以应用在Web, Nodejs, Native。 也可以自定义自己的渲染器。

通过React.Component定义本地组件
```javascript
class CountComponent extends React.Component {
  constructor (props) {
    super(props)

    this.state = {
      count: 0
    }

    this.updateCount = this.updateCount.bind(this)
  }

  componentDidMount () {
    this.updateCount(this.props.num)
  }

  componentDidUpdate (prevProps) {
    if (prevProps.num !== this.props.num) {
      this.updateCount(this.props.num)
    }
  }

  updateCount (num) {
    this.setState({ count: num })
  }

  clickAdd () {
  	this.updateCount(this.state.count + 1)
  }

  render() {
    return (
      <ul>
        <li><a>{this.state.count}</a></li>
        <input type="button" value="Add" onclick="clickAdd()" />
      </ul>
    )
  }
}
```

我们遇到了哪些问题：
1. 具有状态的组件必须用类实现，比较耦合
2. 组件支持函数来创建，但是支持力度相当有限
3. 总是要调用super(props)
4. 有各种this围绕着
5. 逻辑的共享没有合适的手段

Hook的出现很好的解决了这些问题， 而且远不止这些。


## 规则
Hook 是向下兼容的。 Hook的本质就是JavaScript函数， 使用它要遵循两条规则：

### 一、只在最顶层使用Hook
不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层调用他们。
>遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。

### 二、只在React函数中调用Hook
不要在普通的 JavaScript 函数中调用 Hook。

## 其他注意点
### 闭包陷阱
当我们更新状态的时候，React会重新渲染组件。每一次渲染都能拿到独立的count 状态，这个状态值是函数中的一个常量。每一次调用引起的渲染中，它包含的count值独立于其他渲染
```javascript
import React, { useState } from 'react';

function Example() {
	const [count, setCount] = useState(0);

    function alertCount(){
        setTimeout(()=>{
          // alert 只能获取到点击按钮时的那个状态
          alert(count);
        }, 3000);
      }

	return (
	  <div>
	    <p>You clicked {count} times</p>
	    <button onClick={() => setCount(count + 1)}>Click me</button>
        <button onClick={alertCount}>alertCount</button>
	  </div>
	);
}

export default Example;
```
运行一下，先点击1次「alertcount」, 再多点3次「Click me」, 发现 页面显示的是`You clicked 3 times`, 但是alert里显示的是0，和我们预想的不一样。
Hook都有闭包陷阱问题。

原因是：每一次 render 都会生成一个闭包，每个闭包都有自己的 state 和 props。
所以在异步函数中访问hooks的state值拿到的是当前的闭包值，并不是最新的state值。

#### 解决闭包陷阱的办法：
1. 添加count依赖, 缺点是会重复生成和销毁定时器，影响性能
```javascript
useEffect(() => {
    ...
    // 添加count依赖
  }, [count]);
```

2. setCount里换成箭头函数
```javascript
// set函数可以为一个函数(类似于setState) 参数为上一次的state值
setCount(count => count + 1);
```

3. 使用useReducer
```javascript
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'add':
      return {count: state.count + 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'add'})}>+</button>
    </>
  );
}
```

4. 使用useRef
初始化的 useRef 执行之后，返回的都是同一个对象。比如：
```javascript
ref = useRef()
```

组件每一次渲染的过程中，返回的都是同一个对象，每次组件更新所生成的ref指向的都是同一片内存空间，所以每次都能拿到最新的值。