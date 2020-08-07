---
layout:     post
title:      React Hook学习(useEffect)
subtitle:   useEffect的使用与注意点
description: useEffect的使用与注意点
date:        2020-07-24 13:00:00
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

> 如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。

在 React 组件中有两种副作用操作：需要清除的和不需要清除的。

### 不需要清除的effect
有时候，我们只想在 React 更新 DOM 之后运行一些额外的代码。比如发送网络请求，手动变更 DOM，记录日志，这些都是常见的无需清除的操作。因为我们在执行完这些操作之后，就可以忽略他们了。让我们对比一下使用 class 和 Hook 都是怎么实现这些副作用的。

**使用 class 的示例**<br>
在 React 的 class 组件中，render 函数是不应该有任何副作用的。一般来说，在这里执行操作太早了，我们基本上都希望在 React 更新 DOM 之后才执行我们的操作。<br>

这就是为什么在 React class 中，我们把副作用操作放到 componentDidMount 和 componentDidUpdate 函数中。回到示例中，这是一个 React 计数器的 class 组件。它在 React 对 DOM 进行操作之后，立即更新了 document 的 title 属性

```javascript
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
```

我们发现在类组件里需要在两个生命周期里写重复的代码，因为我们希望在加载和更新时执行相同的操作。

我们看一下如何使用useEffect执行相同的操作

**使用 Hook 的示例**
```javascript
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
```

useEffect 跟 class 组件中的 componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。<br>

>useEffect 会在每次`渲染`和`更新`后都执行, React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。
每次我们重新渲染，都会生成新的 effect，替换掉之前的。所以我们每次拿到的count都是最新的。<br>

### 需要清除的effect
一些副作用是需要清除的。比如依赖外部的数据源，这种情况下清除非常有必要，可以防止内存泄漏。

**使用 class 的示例**

比如我们有一个 ChatAPI 模块，它可以订阅好友的在线状态

```javascript
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
```

**使用 Hook 的示例**
```javascript
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
```

这样就可以把添加和移除订阅的逻辑放在一起了。

>为了防止内存泄漏，React 会在组件卸载的时候执行清除操作

## 其它特性
接下来深入了解 useEffect 的某些特性。

### 使用多个 Effect 
使用多个 Effect 可以实现关注点的分离。

下面代码是前面的计数器和好友在线状态指示器的逻辑组合在一起的组件：
在class组件里的示例：
```javascript
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
```

我们可以看到componentDidMount()包含了两个不同功能的代码。
用Hook可以使用多个effect，把不相关的逻辑分离到不同的effect中。

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

React 将按照 effect 声明的顺序依次调用组件中的每一个 effect。

### 每次更新都会运行effect
effect 的清除阶段在每次重新渲染时都会执行, 这将为我们减少bug。

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

当组件已经显示在屏幕上时，props的friend发生变化时，我们的组件将继续展示原来的好友状态，这是一个bug。而且我们还会因为移除订阅错误的好友ID导致内存泄漏或者崩溃的问题。
在class组件里，我们需要用componentDidUpdate来解决这个问题。
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
因为 useEffect 默认就会处理。它会在调用一个新的 effect 之前对前一个 effect 进行清理。

### 添加依赖进行性能优化
在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。在 class 组件中，我们可以通过在 componentDidUpdate 中添加对 prevProps 或 prevState 的比较逻辑解决：
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

上面这个示例中，我们传入 [count] 作为第二个参数。如果 count 的值在组件重渲染的时候 count 值没变化，React 会跳过这个 effect，这就实现了性能的优化。

对于有清除操作的 effect 同样适用。
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

