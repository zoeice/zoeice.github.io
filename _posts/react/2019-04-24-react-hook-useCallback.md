---
layout:     post
title:      React Hook学习(useMemo和useCallback)
subtitle:   useMemo和useCallback的区别和使用场景
description: useMemo和useCallback非常相似，他们的区别是什么？使用场景是什么？
date:        2020-04-24 14:30:00
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

`useCallback` 和 `useMemo` 非常相似，所以放在同一章看。
>useCallback(fn, deps) 相当于 useMemo(() => fn, deps)

那么它们的区别是什么？大概来说呢，区别就是 `useCallback` 缓存函数的引用，`useMemo` 缓存计算数据的值。

## 源码分析

为了清楚一点，我们来看下 `useCallback` 和 `useMemo` 的源码 ReactFiberHooks.new.js：
```js
function updateCallback<T>(
    callback: T, 
    deps: Array<mixed> | void | null
): T {
    const hook = updateWorkInProgressHook();
    const nextDeps = deps === undefined ? null : deps;
    const prevState = hook.memoizedState;
    if (prevState !== null) {
        if (nextDeps !== null) {
            const prevDeps: Array<mixed> | null = prevState[1];
            if (areHookInputsEqual(nextDeps, prevDeps)) {
                return prevState[0];
            }
        }
    }
    hook.memoizedState = [callback, nextDeps];
    return callback;
}

function updateMemo<T>(
    nextCreate: () => T,
    deps: Array<mixed> | void | null,
): T {
    const hook = updateWorkInProgressHook();
    const nextDeps = deps === undefined ? null : deps;
    const prevState = hook.memoizedState;
    if (prevState !== null) {
        // Assume these are defined. If they're not, areHookInputsEqual will warn.
        if (nextDeps !== null) {
            const prevDeps: Array<mixed> | null = prevState[1];
            if (areHookInputsEqual(nextDeps, prevDeps)) {
                return prevState[0];
            }
        }
    }
    const nextValue = nextCreate();
    hook.memoizedState = [nextValue, nextDeps];
    return nextValue;
}

const HooksDispatcherOnUpdate: Dispatcher = {
    useCallback: updateCallback,
    useMemo: updateMemo,
    ...
};
```

细心的已经可以看出二者的不同之处了。<br>

1. 看下 `updateCallback` 里这句：

```js
hook.memoizedState = [callback, nextDeps];
```

**缓存的是传入函数的引用。**<br>
这样有什么用处呢？<br>
- 如果依赖项没有变化，走的就缓存函数的逻辑。<br>
- 只有依赖项变化，才会重新调用新的函数。<br>

2. 看下 `updateMemo` 里这句：

```js
hook.memoizedState = [nextValue, nextDeps];
```

**缓存的是的 传入函数nextCreate()的返回值。**<br>
- 如果依赖项没有变化，是不会执行计算函数的，直接用缓存的值。<br>
- 只有依赖项变化才会调用重新计算函数。

## 应用场景

### useCallback应用
先找一个父子组件的例子，先写parent.js：
```javascript
import React, { useState } from 'react';
import Child from './child';

const Parent = () => {
    const [count, setCount] = useState(0);
    const clickChild = () => {
        console.log('click Child');
    };

    const clickParent = () => {
        console.log('click Parent');
        setCount(preCount => preCount + 1);
    };

    return (
        <div>
            Count: {count}
            <hr />
            <button onClick={clickParent}>click Parent</button>
            <Child clickChild={clickChild} />
        </div>
    );
};

export default Parent;
```

然后是child.js
```javascript
import React, { memo } from 'react';

const Child = memo(({ clickChild }) => {
    console.log('Child render >>');
    return <button onClick={clickChild}>clickChild</button>;
});

export default Child;
```

我们发现，点击2次「click Parent」按钮，控制台会出现：
```shell
click Parent
Child render >>
click Parent
Child render >>
```

发现虽然采用了memo优化，但是每次Parent重新渲染的时候，Child也跟着重新渲染了，这多浪费资源。 <br>
来使用useCallback做如下优化Child的渲染，我们改下parent.js：
```javascript
import React, { useState, useCallback } from 'react';
import Child from './child';

const Parent = () => {
    const [count, setCount] = useState(0);
    // 代码只改了这里哦 >>
    const clickChild = useCallback(() => {
        console.log('click Child');
    }, []);

    const clickParent = () => {
        console.log('click Parent');
        setCount(preCount => preCount + 1);
    };

    return (
        <div>
            Count: {count}
            <hr />
            <button onClick={clickParent}>click Parent</button>
            <Child clickChild={clickChild} />
        </div>
    );
};

export default Parent;
```

再次运行点击2次「click Parent」按钮，看下日志：
```shell
click Parent
click Parent
```

已经不会重新渲染Child组件了，性能得到了提升。<br>
>useCallback 第2个参数是依赖项，加入依赖项后可以控制在那个参数变化后，渲染Child。这里就不展开了。

### useMemo应用
先找个例子看看没使用useMemo的情况：
```js
import React, { useState } from 'react';

const Compute = () => {
    const [count, setCount] = useState(666);
    const [other, setOther] = useState(1);
  
    function computeCount(count) {
        console.log('假设耗时的计算 >>', count);
        return count * count
    }

    const clickUpdateCount = () => {
        setCount(preCount => preCount * 2);
    };

    const clickUpdateOther = () => {
        setOther(other => other + 1);
    };

    const computeValue = computeCount(count);

    return (
        <div>
            Count: {count}
            <br />
            Other: {other}
            <br />
            Result: {computeValue}
            <hr />
            <button onClick={clickUpdateCount}>addCount</button>
            <button onClick={clickUpdateOther}>addOther</button>
        </div>
    );
};

export default Compute;
```

`computeCount(count){}` 的入参是count<br>
点击「addCount」按钮为改变count, 会执行到 `computeCount()`<br>
但是，点击「addOther」按钮，和count没有关系，也会执行到 `computeCount()` ，这就有多点累赘了，还浪费资源影响性能。

我们用useMemo改造一下：
```js
import React, { useState, useMemo } from 'react';

const Compute = () => {
    const [count, setCount] = useState(666);
    const [other, setOther] = useState(1);
  
    function computeCount(count) {
        console.log('假设耗时的计算 >>', count);
        return count * count
    }

    const clickUpdateCount = () => {
        setCount(preCount => preCount * 2);
    };

    const clickUpdateOther = () => {
        setOther(other => other + 1);
    };

    const computeValue = useMemo(() => computeCount(count), [count]);

    return (
        <div>
            Count: {count}
            <br />
            Other: {other}
            <br />
            Result: {computeValue}
            <hr />
            <button onClick={clickUpdateCount}>addCount</button>
            <button onClick={clickUpdateOther}>addOther</button>
        </div>
    );
};

export default Compute;
```

运行下看看效果：<br>
点击「addCount」按钮为改变count, 会执行到 `computeCount()`<br>
但是，点击「addOther」按钮，不会执行到 `computeCount()` ，性能得以提升。

## 其他思考
### 如何测量 DOM 节点？
获取 DOM 节点的位置或是大小的基本方式是使用 callback ref。每当 ref 被附加到一个另一个节点，React 就会调用 callback。这里有一个 小 demo:

```js
function MeasureExample() {
    const [height, setHeight] = useState(0);

    const measuredRef = useCallback(node => {
        if (node !== null) {
            setHeight(node.getBoundingClientRect().height);
        }
    }, []);

    return (
        <>
            <h1 ref={measuredRef}>Hello, world</h1>
            <h2>The above header is {Math.round(height)}px tall</h2>
        </>
    );
}
```

在这个案例中，我们没有选择使用 useRef，因为当 ref 是一个对象时它并不会把当前 ref 的值的 变化 通知到我们。使用 callback ref 可以确保 即便子组件延迟显示被测量的节点 (比如为了响应一次点击)，我们依然能够在父组件接收到相关的信息，以便更新测量结果。

注意到我们传递了 [] 作为 useCallback 的依赖列表。这确保了 ref callback 不会在再次渲染时改变，因此 React 不会在非必要的时候调用它。

>在此示例中，仅在安装和卸载组件时才调用回调ref，因为渲染的<h1>组件在所有重新渲染期间均保持存在。 如果您希望在组件调整大小时收到通知，则可能需要使用ResizeObserver或基于其构建的第三方Hook。


如果你愿意，你可以 把这个逻辑抽取出来作为 一个可复用的 Hook:
```js
function MeasureExample() {
    const [rect, ref] = useClientRect();
    return (
        <>
            <h1 ref={ref}>Hello, world</h1>
            {rect !== null &&
                <h2>The above header is {Math.round(rect.height)}px tall</h2>
            }
        </>
    );
}

function useClientRect() {
    const [rect, setRect] = useState(null);
    const ref = useCallback(node => {
        if (node !== null) {
            setRect(node.getBoundingClientRect());
        }
    }, []);
    return [rect, ref];
}
```