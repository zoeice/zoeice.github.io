---
layout:     post
title:      React Hook学习(useContext)
subtitle:   useContext的使用与注意点
description: useContext的使用与注意点
date:        2020-07-24 12:00:00
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
子组件中修改父组件的 state<br>
一般的做法是将父组件的方法比如 setXXX 通过 props 的方式传给子组件，而一旦子组件多层级的话，就要层层透传。<br>
使用 Context 的方式则可以免去这种层层透传

## 如何使用
### 创建Context
createContext 能能够创建一个 React 的 context, 然后再订阅了这个 context 的组件中，可以拿到 context 提供的数据或者替他信息。

```javascript
// createContext的参数是传入的默认值，匹配不到最新的Provider时会用到默认值。
const MyContext = React.createContext(defaultValue)
```

如果要使用创建的 context，需要通过 Context.Provider 在最外层包装组件，并且通过 `<MyContext.Provider value={{a:b}}>` 的方式传入 value，指定 context 要对外提供的信息。

子组件在匹配过程中只会匹配最新的 Provider
比如在这样的结构中：
ContextA.Provider/组件A/ContextB.Provider/组件B/组件C
那么组件C只能拿到ContextB的数据

### 使用Context
根据 [官方useContext文档](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext) 看到：
子组件可以通过 useContext 这个Hook获取到 Provider 提供的内容。
```javascript
const value = useContext(MyContext);
```

它有这几个特征：<br>
1. 接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

2. 调用了 useContext 的组件总会在 context 值变化时重新渲染。并使用最新的context值。

3. 即使祖先使用 React.memo 或 shouldComponentUpdate，也会在组件本身使用 useContext 时重新渲染。

4. useContext(MyContext) 相当于 class 组件中的 static contextType = MyContext 或者 <MyContext.Consumer>

5. useContext(MyContext) 只是让你能够读取 context 的值以及订阅 context 的变化。你仍然需要在上层组件树中使用 <MyContext.Provider> 来为下层组件提供 context

不过细心的我们发现：<br>
MyContext 是需要传入的实例，但是从哪里获取呢？Provider 一般和子组件不在一个文件。那就需要实现个三方管理器来管理context实例了。

父子组件都从该管理器拿到context的实例。

### 创建和使用context的结合

创建一个context管理器 context-manager.js：
```javascript
import React from 'react';

export const MyContext = React.createContext(null);
```

父组件引入了实例，并且通过 MyContext.Provider 将父组件包装，并且通过 Provider.value 将方法提供出去。
创建父组件 parent.js:
```javascript
import React, { useState } from 'react';
import Child from './child';
import { MyContext } from './context-manager';

function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(1);
        })
    });
}

export default function Parent(props = {}) {
    const [count, setCount] = useState(0);

    return (
        <MyContext.Provider value={{ setCount, fetchData }}>
            <Child count={count} />
        </MyContext.Provider>
    );
}
```

然后是创建子组件 child.js:
```javascript
import React, { useContext, useEffect, memo } from 'react';

import { MyContext } from './context-manager';

export default function Child(props = {}){
    const { setCount, fetchData } = useContext(MyContext);

    useEffect(() => {
        fetchData().then((res) => {
            console.log(`get data >> ${res}`);
        })
    }, []);

    return (
        <div>
            <p>count is : {props.count}</p>
            <hr />
            <div>
                <button onClick={() => { setCount(props.count + 1) }}>Add</button>
            </div>
        </div>
    );
};
```

运行发现，在子组件中点击按钮，直接调用 Context 透传过来的方法，可以修改父组件中的 state, 修改后，子组件也会重新渲染。
这种方式避免了多级 props 层层传递的问题。
