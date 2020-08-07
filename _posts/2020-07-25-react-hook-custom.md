---
layout:     post
title:      React Hook学习(自定义Hook)
subtitle:   如何自定义 Hook 以及注意点
description: 通过自定义 Hook，可以将组件逻辑提取到可重用的函数中
date:        2020-07-24 17:00:00
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

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

我们先找下useEffect里聊天显示好友在线状态的例子：
```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
    const [isOnline, setIsOnline] = useState(null);

    useEffect(() => {
        function handleStatusChange(status) {
            setIsOnline(status.isOnline);
        }
        ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
        return function cleanup() {
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

假设我们还需要一个联系人列表，当用户在线时需要把名字设成绿色。
```js
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
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

    return (
        <li style={{ color: isOnline ? 'green' : 'black' }}>
            {props.friend.name}
        </li>
    );
}
```

两个组件有一些共同的逻辑，我们希望把它们抽离出来共享逻辑，我们看下Hook如何在不增加组件的情况下解决该问题：

## 提取自定义Hook
我们想要共享逻辑，把它提取到第三个函数中，

**自定义Hook是一个函数，其名称已`use`开头，函数内部可以调用其他Hook.

```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
    const [isOnline, setIsOnline] = useState(null);

    useEffect(() => {
        function handleStatusChange(status) {
            setIsOnline(status.isOnline);
        }

        ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
        return () => {
            ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
        };
    });

    return isOnline;
}
```

## 使用自定义Hook
```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
    const isOnline = useFriendStatus(props.friend.id);

    if (isOnline === null) {
        return 'Loading...';
    }
    return isOnline ? 'Online' : 'Offline';
}
export default FriendStatus;
```

```js
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
    const isOnline = useFriendStatus(props.friend.id);

    return (
        <li style={{ color: isOnline ? 'green' : 'black' }}>
            {props.friend.name}
        </li>
    );
}
export default FriendListItem;
```

这段代码完全和上面一致。自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。

## 问题
**自定义 Hook 必须以 “use” 开头吗？**<br>
必须如此。这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 Hook 的规则。<br><br>

**在两个组件中使用相同的 Hook 会共享 state 吗？**<br>
不会。自定义 Hook 是一种重用状态逻辑的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。<br><br>

**自定义 Hook 如何获取独立的 state？**<br>
每次调用 Hook，它都会获取独立的 state。由于我们直接调用了 useFriendStatus，从 React 的角度来看，我们的组件只是调用了 useState 和 useEffect。 正如我们在之前章节中了解到的一样，我们可以在一个组件中多次调用 useState 和 useEffect，它们是完全独立的。<br><br>

## 使用注意
自定义 Hook 解决了以前在 React 组件中无法灵活共享逻辑的问题。你可以创建涵盖各种场景的自定义 Hook，如表单处理、动画、订阅声明、计时器，甚至可能还有其他我们没想到的场景。更重要的是，创建自定义 Hook 就像使用 React 内置的功能一样简单。<br><br>

尽量避免过早地增加抽象逻辑。既然函数组件能够做的更多，那么代码库中函数组件的代码行数可能会剧增。这属于正常现象 —— 不必立即将它们拆分为 Hook。但我们仍鼓励你能通过自定义 Hook 寻找可能，以达到简化代码逻辑，解决组件杂乱无章的目的。