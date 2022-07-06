---
layout:     post
title:      React Hook学习(useEffect)
subtitle:   useEffect的使用与注意点
description: useEffect的使用与注意点
date:        2019-04-25 13:00:00
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
**useEffect** 可以让你在函数组件中执行副作用操作。

> 如果你熟悉 React class 的生命周期函数，你可以把 `useEffect` Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。

在 React 组件中有两种副作用操作：需要清除的和不需要清除的。

### 不需要清除的effect
有时候，我们只想在 React 更新 DOM 之后运行一些额外的代码。比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。因为我们在执行完这些操作之后，就可以忽略他们了。让我们对比一下使用 class 和 Hook 都是怎么实现这些副作用的。

**使用 class 的示例**<br>
在 React 的 class 组件中，render 函数是不应该有任何副作用的。一般来说，在这里执行操作太早了，我们基本上都希望在 React 更新 DOM 之后才执行我们的操作。<br>

这就是为什么在 React class 中，我们把副作用操作放到 `componentDidMount` 和 `componentDidUpdate` 函数中。回到示例中，这是一个 React 计数器的 class 组件。它在 React 对 DOM 进行操作之后，立即更新了 document 的 title 属性

~~~js
import React from 'react'

class Example extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        };
    }

    componentDidMount() {
        document.title = `You clicked ${this.state.count} times`;
    }
    componentDidUpdate() {
        document.title = `You clicked ${this.state.count} times`;
    }

    render() {
        return (
            <div>
                <p>You clicked {this.state.count} times</p>
                <button onClick={() => this.setState({ count: this.state.count + 1 })}>
                    Click me
                </button>
            </div>
        );
    }
}
export default Example;
~~~
我们发现在类组件里需要在两个生命周期里写重复的代码，因为我们希望在加载和更新时执行相同的操作。

我们看一下如何使用 `useEffect` 执行相同的操作

**使用 Hook 的示例**
~~~js
import React, { useState, useEffect } from 'react';

function Example() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        document.title = `You clicked ${count} times`;
    });

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={() => setCount(count + 1)}>
                Click me
            </button>
        </div>
    );
}
export default Example;
~~~

`useEffect` 跟 class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API。<br>

>`useEffect` 会在每次`渲染`和`更新`后都执行, React 保证了每次运行 `effect` 的同时，DOM 都已经更新完毕。
每次我们重新渲染，都会生成新的 `effect`，替换掉之前的。所以我们每次拿到的 `count` 都是最新的。<br>

### 需要清除的effect
一些副作用是需要清除的。比如依赖外部的数据源，这种情况下清除非常有必要，可以防止内存泄漏。

**使用 class 的示例**

比如我们有一个 ChatAPI 模块，它可以订阅好友的在线状态

~~~js
class FriendStatus extends React.Component {
    constructor(props) {
        super(props);
        this.state = { isOnline: null };
        this.handleStatusChange = this.handleStatusChange.bind(this);
    }

    componentDidMount() {
        // 订阅好友的状态
        ChatAPI.subscribeToFriendStatus(
            this.props.friend.id,
            this.handleStatusChange
        );
    }
    componentWillUnmount() {
        // 移除订阅
        ChatAPI.unsubscribeFromFriendStatus(
            this.props.friend.id,
            this.handleStatusChange
        );
    }
    handleStatusChange(status) {
        this.setState({
            isOnline: status.isOnline
        });
    }

    render() {
        if (this.state.isOnline === null) {
            return 'Loading...';
        }
        return this.state.isOnline ? 'Online' : 'Offline';
    }
}
export default FriendStatus;
~~~

**使用 Hook 的示例**
~~~js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
    const [isOnline, setIsOnline] = useState(null);

    useEffect(() => {
        function handleStatusChange(status) {
            setIsOnline(status.isOnline);
        }
        // 订阅好友的状态
        ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
        // 指定此效果后如何清理
        return function cleanup() {
            // 移除订阅
            ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
        };
    });

    if (isOnline === null) {
        return 'Loading...';
    }
    return isOnline ? 'Online' : 'Offline';
}
export default FriendStatus;
~~~

这样就可以把添加和移除订阅的逻辑放在一起了。

>为了防止内存泄漏，React 会在组件卸载的时候执行清除操作

## 其它特性
接下来深入了解 `useEffect` 的某些特性。

### 使用多个 Effect 
使用多个 `Effect` 可以实现关注点的分离。

下面代码是前面的计数器和好友在线状态指示器的逻辑组合在一起的组件：
在class组件里的示例：
~~~js
class FriendStatusWithCounter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0, isOnline: null };
        this.handleStatusChange = this.handleStatusChange.bind(this);
    }

    componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
        ChatAPI.subscribeToFriendStatus(
            this.props.friend.id,
            this.handleStatusChange
        );
    }

    componentDidUpdate() {
        document.title = `You clicked ${this.state.count} times`;
    }

    componentWillUnmount() {
        ChatAPI.unsubscribeFromFriendStatus(
            this.props.friend.id,
            this.handleStatusChange
        );
    }

    handleStatusChange(status) {
        this.setState({
            isOnline: status.isOnline
        });
    }

    render() {
        if (this.state.isOnline === null) {
            return 'Loading...';
        }
        return (
            <div>
                { this.state.isOnline ? 'Online' : 'Offline' }
                <p>You clicked {this.state.count} times</p>
                <button onClick={() => this.setState({ count: this.state.count + 1 })}>
                    Click me
                </button>
            </div>
        );
    }
}
export default Example;
~~~

我们可以看到 `componentDidMount()` 包含了两个不同功能的代码。
用Hook可以使用多个 `effect`，把不相关的逻辑分离到不同的 `effect` 中。

```javascript
function FriendStatusWithCounter(props) {
    const [count, setCount] = useState(0);
    useEffect(() => {
        document.title = `You clicked ${count} times`;
    });

    const [isOnline, setIsOnline] = useState(null);
    useEffect(() => {
        function handleStatusChange(status) {
            setIsOnline(status.isOnline);
        }
        ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
        return () => {
            ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
        };
    });

    if (this.state.isOnline === null) {
        return 'Loading...';
    }
    return (
        <div>
            { this.state.isOnline ? 'Online' : 'Offline' }
            <p>You clicked {this.state.count} times</p>
            <button onClick={() => this.setState({ count: this.state.count + 1 })}>
                Click me
            </button>
        </div>
    );
}
export default Example;
```

React 将按照 `effect` 声明的顺序依次调用组件中的每一个 `effect。`

### 每次更新都会运行effect
`effect` 的清除阶段在每次重新渲染时都会执行, 这将为我们减少bug。

比如前面这个例子：
```javascript
componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}

componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}
```

当组件已经显示在屏幕上时，`props` 的friend发生变化时，我们的组件将继续展示原来的好友状态，这是一个bug。而且我们还会因为移除订阅错误的好友ID导致内存泄漏或者崩溃的问题。
在class组件里，我们需要用 `componentDidUpdate` 来解决这个问题。
```javascript
componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}

componentDidUpdate(prevProps) {
    // 移除订阅之前的 friend.id
    ChatAPI.unsubscribeFromFriendStatus(
        prevProps.friend.id,
        this.handleStatusChange
    );
    // 订阅新的 friend.id
    ChatAPI.subscribeToFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}

componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}
```

现在看一下 Hook 的版本：
```javascript
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
});
```
虽然我们没做任何变动，但它并不会出现该bug。
因为 `useEffect` 默认就会处理。它会在调用一个新的 `effect` 之前对前一个 `effect` 进行清理。

### 添加依赖进行性能优化
在某些情况下，每次渲染后都执行清理或者执行 `effect` 可能会导致性能问题。在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：
```javascript
componentDidUpdate(prevProps, prevState) {
    if (prevState.count !== this.state.count) {
        document.title = `You clicked ${this.state.count} times`;
    }
}
```

在 Hook 中，你只要传递数组作为 useEffect 的第二个可选参数即可：
```javascript
useEffect(() => {
    document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```

上面这个示例中，我们传入 [count] 作为第二个参数。如果 `count` 的值在组件重渲染的时候 `count` 值没变化，React 会跳过这个 `effect`，这就实现了性能的优化。

对于有清除操作的 `effect` 同样适用。
```javascript
useEffect(() => {
    function handleStatusChange(status) {
        setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
}, [props.friend.id]); // 仅在 props.friend.id 发生变化时，重新订阅
```

## 其他思考
### 在依赖列表中省略函数是否安全？
一般来说，不安全。
```js
function Example({ someProp }) {
    function doSomething() {
        console.log(someProp);
    }

    useEffect(() => {
        doSomething();
    }, []); // 🔴 这样不安全（它调用的 `doSomething` 函数使用了 `someProp`）
}
```

要记住 `effect` 外部的函数使用了哪些 `props` 和 `state` 很难。这也是为什么 通常你会想要在 `effect` 内部 去声明它所需要的函数。 这样就能容易的看出那个 `effect` 依赖了组件作用域中的哪些值：

```js
function Example({ someProp }) {
    useEffect(() => {
        function doSomething() {
            console.log(someProp);
        }

        doSomething();
    }, [someProp]); // ✅ 安全（我们的 effect 仅用到了 `someProp`）
}
```

如果这样之后我们依然没用到组件作用域中的任何值，就可以安全地把它指定为 []：

```js
useEffect(() => {
    function doSomething() {
        console.log('hello');
    }

    doSomething();
}, []); // ✅ 在这个例子中是安全的，因为我们没有用到组件作用域中的 *任何* 值
```

### 函数适合放在 effect 内部还是外部？
我们来看一个例子：
```js
function ProductPage({ productId }) {
    const [product, setProduct] = useState(null);

    async function fetchProduct() {
        const response = await fetch('http://myapi/product/' + productId); // 使用了 productId prop
        const json = await response.json();
        setProduct(json);
    }

    useEffect(() => {
        fetchProduct();
    }, [productId]); // ✅ 有效，因为我们的 effect 只用到了 productId
    // ...
}
```

虽然看起来确实没啥问题，但是不是很直观的能看到函数和依赖项的关系。<br>
推荐的方案是把那个函数移动到你的 `effect` 内部
```js
function ProductPage({ productId }) {
    const [product, setProduct] = useState(null);

    useEffect(() => {
        // 把这个函数移动到 effect 内部后，我们可以清楚地看到它用到的值。
        async function fetchProduct() {
        const response = await fetch('http://myapi/product/' + productId);
        const json = await response.json();
        setProduct(json);
        }

        fetchProduct();
    }, [productId]); // ✅ 有效，因为我们的 effect 只用到了 productId
    // ...
}
```

这样就很容易看出你的 `effect` 使用了哪些 `props` 和 `state`，并确保他们都被声明了。

如果处于某些原因你 无法 把一个函数移动到 `effect` 内部，还有一些其他办法：
- 你可以尝试把那个函数移动到你的组件之外。
- 如果你所调用的方法是一个纯计算，并且可以在渲染时调用，你可以 转而在 effect 之外调用它， 并让 effect 依赖于它的返回值。
- 万不得已的情况下，你可以 把函数加入 `effect` 的依赖但 把它的定义包裹 进 `useCallback` Hook。

```js
function ProductPage({ productId }) {
    // ✅ 用 useCallback 包裹以避免随渲染发生改变
    const fetchProduct = useCallback(() => {
        // ... Does something with productId ...
    }, [productId]); // ✅ useCallback 的所有依赖都被指定了

    return <ProductDetails fetchProduct={fetchProduct} />;
}

function ProductDetails({ fetchProduct }) {
    useEffect(() => {
        fetchProduct();
    }, [fetchProduct]); // ✅ useEffect 的所有依赖都被指定了
    // ...
}
```

注意在上面的案例中，我们 需要 让函数出现在依赖列表中。这确保了 ProductPage 的 productId prop 的变化会自动触发 ProductDetails 的重新获取。

### effect 的依赖项频繁变化怎么办？
```js
function Counter() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const id = setInterval(() => {
            setCount(count + 1); // 这个 effect 依赖于 `count` state
        }, 1000);
        return () => clearInterval(id);
    }, [count]); // ✅ useEffect 的所有依赖都被指定了

    return <h1>{count}</h1>;
}
```

count被加进依赖项，每次变化都会导致定时器被重置。这并不是我们想要的。要解决这个问题，我们可以使用 `setState` 函数式的更新形式，它允许我们指定 `state` 该 如何 改变而不用引用 当前 `state`：
```js
function Counter() {
    const [count, setCount] = useState(0);

    useEffect(() => {
        const id = setInterval(() => {
            setCount(c => c + 1); // ✅ 在这不依赖于外部的 `count` 变量
        }, 1000);
        return () => clearInterval(id);
    }, []); // ✅ 我们的 effect 不适用组件作用域中的任何变量

    return <h1>{count}</h1>;
}
```

此时，`setInterval` 的回调依旧每秒调用一次，但每次 `setCount` 内部的回调取到的 `count` 是最新值（在回调中变量命名为 c）。

在一些更加复杂的场景中（比如一个 state 依赖于另一个 state），尝试用 `useReducer` Hook 把 `state` 更新逻辑移到 `effect` 之外。
>`useReducer` 的 `dispatch` 的身份永远是稳定的 —— 即使 `reducer` 函数是定义在组件内部并且依赖 `props`。