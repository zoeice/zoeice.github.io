---
layout:     post
title:      React Hook学习(useRef)
subtitle:   useRef的区别和使用场景
description: useRef的区别和使用场景
date:        2020-07-24 15:00:00
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
类组件、React 元素用 React.createRef，函数组件使用 useRef

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。

```js
const refContainer = useRef(initialValue);
```

## 用途
- useRef可以获取组件的实例
- useRef会保持引用不变，ref.current的值改变不会引起组件重新渲染，类似类组件的this.xxx

### 拿到组件实例
先找一个例子来看下使用：
```js
function TextInputWithFocusButton() {
    const inputEl = useRef(null);
    const onButtonClick = () => {
        // `current` 指向已挂载到 DOM 上的文本输入元素
        inputEl.current.focus();
    };
    return (
        <>
            <input ref={inputEl} type="text" />
            <button onClick={onButtonClick}>Focus the input</button>
        </>
    );
}
```

本质上，`useRef` 就像是可以在其 `.current` 属性中保存一个可变值的“盒子”。<br><br>

你应该熟悉 ref 这一种访问 DOM 的主要方式。如果你将 ref 对象以 `<div ref={myRef} />` 形式传入组件，则无论该节点如何改变，React 都会将 `ref` 对象的 `.current` 属性设置为相应的 DOM 节点。<br><br>

然而，`useRef()` 比 `ref` 属性更有用。它可以很方便地保存任何可变值，其类似于在 class 中使用实例字段的方式。<br><br>

这是因为它创建的是一个普通 Javascript 对象。而 `useRef()` 和自建一个 `{current: ...}` 对象的唯一区别是，`useRef` 会在每次渲染时返回同一个 `ref` 对象。<br><br>

请记住，当 `ref` 对象内容发生变化时，`useRef` 并不会通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref 来实现。


### 区分首次渲染还是后续渲染
你可以创建一个可变的ref存储布尔值来区分组件是首次渲染还是后续的渲染
```js
import React, { useState, useRef } from 'react';

const Example = () => {
    const [count, setCount] = useState(0);
  
    const prevCountRef = useRef(false);
    useEffect(() => {
        if (prevCountRef.current) {
            console.log('只在更新时候执行, 可以在这里加逻辑');
        } else {
            prevCountRef.current = true;
        }
    });
  
    return (
        <div>
            <div>{count}</div>
            <button onClick={() => { setCount(count + 1) }}>Add</button>
        </div>
    )
}

export default Example;
```

可以封装一下让它更简单：
```js
import React, { useState, useRef, useEffect } from 'react';

const Example = () => {
    const [count, setCount] = useState(0);
  
    const reRender = useReRender()
    if(reRender) {
        console.log('后续渲染')
    } else {
        console.log('首次渲染')
    }
  
    return (
        <div>
            <div>{count}</div>
            <button onClick={() => { setCount(count + 1) }}>Add</button>
        </div>
    )
}

function useReRender() {
    const ref = useRef(false)
    useEffect(() => {
        ref.current = true
    })
    return ref.current
}

export default Example;
```

### 获取上一次的props和state
也可以通过ref来实现：
```js
import React, { useState, useRef, useEffect } from 'react';

const Example = (props = {}) => {
    const [count, setCount] = useState(0);
  
    const prevPropsRef = useRef();
    useEffect(() => {
        prevPropsRef.current = props
    });

    const prevCountRef = useRef();
    useEffect(() => {
        prevCountRef.current = count
    });
  
    return (
        <div>
            <div>{count}</div>
            <button onClick={() => { setCount(count + 1) }}>Add</button>
        </div>
    )
}

export default Example;
```

为了方便使用，我们也把它封装一下：
```js
import React, { useState, useRef, useEffect } from 'react';

const Example = (props = {}) => {
    const [count, setCount] = useState(0);
  
    const prevProps = usePrevious(props)
    console.log('拿到props之前的状态', prevProps)
    const prevCount = usePrevious(count)
    console.log('拿到count之前的状态', prevCount)
  
    return (
        <div>
            <div>{count}</div>
            <button onClick={() => { setCount(count + 1) }}>Add</button>
        </div>
    )
}

function usePrevious (value) {
    const ref = useRef()
    useEffect(() => {
        ref.current = value
    })
    return ref.current
}

export default Example;
```