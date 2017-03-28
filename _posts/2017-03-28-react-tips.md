---
layout: post
title: React 一叶知秋
keywords: react, 源码, tips, setState, ReactEventListener
description: React 一叶知秋（不断整理更新...）
date: '2017-03-28 11:00:00 +0800'
categories: react
---

# React 一叶知秋


## React 官方建议不要进行 DOM 节点跨层级的操作

React 对树的算法进行了简洁明了的优化，即对树进行分层比较，两棵树只会对同一层次的节点进行比较。若出现 DOM 节点跨层级的移动操作时，React diff 会将所移动节点的树重新创建，这是一种影响 React 性能的操作。所以在开发组件时，**保持稳定的 DOM 结构会有助于性能的提升**。例如，可以通过 CSS 隐藏或显示节点，而不是真的移除或添加 DOM 节点。


## setState 及事件系统

setState 会触发 batchedUpdate，那么触发时机或者整个流程是怎样的呢？

跟踪源码会发现，setState 实际上执行的是一个入队的操作，具体可参见文件 [React 源码概览二 （渲染模块）](http://front-ender.me/react/react-source-code-render.html)

那么如何执行这些更新操作呢？文章 [React Transaction 机制](http://front-ender.me/react/react-transaction.html) 中的一幅图已经介绍的很详细了。

![react_transaction](../resources/react_transaction.png)

执行的起点是 ReactEventListener.dispatchEvent 方法，何时才会触发该操作呢？

这里就不得不提 React 的事件系统了，会将所有事件都自动绑定到最外层，并维护一个保存了所有组件内部事件监听及处理函数的映射，最终使用统一的事件监听器对事件进行处理。普通开发者并不需要关心这一层，除非你有特殊的需求。

React 中事件系统的初始化过程如下：

```javascript
ReactDefaultInjection.inject()  =>

  ReactInjection.EventEmitter.injectReactEventListener(ReactEventListener)  =>

    ┌ ReactEventListener.trapBubbledEvent   ┐
    ┤                                       ├ =>
    └ ReactEventListener.trapCapturedEvent  ┘

      EventListener.capture(element, handlerBaseName, ReactEventListener.dispatchEvent.bind(null, topLevelType));
```

其中，`ReactEventListener.dispatchEvent` 即使注册的叶子节点，同样也是事件响应的起点，这样，整个流程就串联起来了。

## 参考资料
