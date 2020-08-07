---
layout:     post
title:      React Hook学习(useReducer)
subtitle:   useReducer的使用与注意点
description: useReducer的使用与注意点
date:        2020-07-24 14:00:00
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
`useReducer` 是 `useState` 的替代方案。有如下特性：

- 和redux 中 `reducer` 很像
- `useState` 内部就是靠 `useReducer` 来实现的
- 在某些场景下，`useReducer` 会比 `useState` 更适用，例如 `state` 逻辑较复杂且包含多个子值，或者下一个 `state` 依赖于之前的 state 等。
- 使用 `useReducer` 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 `dispatch` 而不是回调函数 。

useReducer的函数是这样的：
```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

## 示例

在setState里的例子是：
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

如果用 `useReducer` 来实现是什么样子的：
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

function Example() {
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <>
            Count: {state.count}
            <button onClick={() => dispatch({type: 'add'})}>Add</button>
        </>
    );
}

export default Example;
```

>注意：<br>React 会确保 `dispatch` 函数的标识是稳定的，并且不会在组件重新渲染时改变。这就是为什么可以安全地从 `useEffect` 或 `useCallback` 的依赖列表中省略 `dispatch`。

## 指定初始化
有两种不同初始化 `useReducer` `state` 的方式，你可以根据使用场景选择其中的一种。将初始 `state` 作为第二个参数传入 `useReducer` 是最简单的方法：
```javascript
const [state, dispatch] = useReducer(
    reducer,
    {count: initialCount}
);
```

## 惰性初始化
你可以选择惰性地创建初始 `state`。为此，需要将 `init` 函数作为 `useReducer` 的第三个参数传入，这样初始 `state` 将被设置为 `init(initialArg)`。

这么做可以将用于计算 `state` 的逻辑提取到 `reducer` 外部，这也为将来对重置 `state` 的 `action` 做处理提供了便利：

```javascript
function init(initialCount) {
    return {count: initialCount};
}

function reducer(state, action) {
    switch (action.type) {
        case 'add':
            return {count: state.count + 1};
        case 'reset':
            return init(action.payload);
        default:
            throw new Error();
    }
}

function Example({initialCount}) {
    const [state, dispatch] = useReducer(reducer, initialCount, init);
    return (
        <>
            Count: {state.count}
            <button onClick={() => dispatch({type: 'reset', payload: initialCount})}>Reset</button>
            <button onClick={() => dispatch({type: 'add'})}>Add</button>
        </>
    );
}
    
export default Example;
```

## 跳过dispatch
如果 `Reducer` Hook 的返回值与当前 `state` 相同，React 将跳过子组件的渲染及副作用的执行。（React 使用 Object.is 比较算法 来比较 `state`。）

需要注意的是，React 可能仍需要在跳过渲染前再次渲染该组件。不过由于 React 不会对组件树的“深层”节点进行不必要的渲染，所以大可不必担心。如果你在渲染期间执行了高开销的计算，则可以使用 `useMemo` 来进行优化。

