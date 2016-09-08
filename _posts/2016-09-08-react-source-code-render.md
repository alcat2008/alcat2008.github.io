---
layout: post
title: React 源码概览二 （渲染模块）
keywords: react, 源码, 概览, overview, 渲染
description: React 源码概览二 （渲染模块）
date: '2016-09-08 11:00:00 +0800'
categories: react
---

# React 源码概览二 （渲染模块）

上一篇文章 [React 源码概览一 （核心接口）](/react/react-source-code-core.html) 介绍了 React 源码的核心接口部分，本文继续介绍 React 源码中另一非常重要的模块：渲染。

React.js 定义了通用的模型，但并没有强制限定底层是用 DOM 还是 React Native 渲染，这也是能支持全平台通用的基石。不得不佩服 FB 人的架构能力，下面就介绍其渲染逻辑。

## renderers 渲染逻辑

先介绍两个关键的工具： ReactUpdates 和 ReactReconciler。

### ReactUpdates

ReactUpdates 提供了批量更新的公共方法。

```javascript
var ReactUpdates = {
  ReactReconcileTransaction: null,

  batchedUpdates: batchedUpdates,
  enqueueUpdate: enqueueUpdate,
  flushBatchedUpdates: flushBatchedUpdates,
  injection: ReactUpdatesInjection,
  asap: asap,
};
```

### ReactReconciler

ReactReconciler 提供了控制渲染流程的方法。

```javascript
var ReactReconciler = {
  // 组件初始化，渲染标记，注册事件监听器
  mountComponent: function(
    internalInstance,
    transaction,
    hostParent,
    hostContainerInfo,
    context,
    parentDebugID // 0 in production and for roots
  ) {
    var markup = internalInstance.mountComponent(
      transaction,
      hostParent,
      hostContainerInfo,
      context,
      parentDebugID
    );
    if (internalInstance._currentElement &&
        internalInstance._currentElement.ref != null) {
      transaction.getReactMountReady().enqueue(attachRefs, internalInstance);
    }
    return markup;
  },

  // 得到可以传递给 ReactComponentEnvironment.replaceNodeWithMarkup 的值
  getHostNode: function(internalInstance) {
    return internalInstance.getHostNode();
  },

  // 释放 `mountComponent` 分配的资源
  unmountComponent: function(internalInstance, safely) {
    ReactRef.detachRefs(internalInstance, internalInstance._currentElement);
    internalInstance.unmountComponent(safely);
  },

  // 用新的 element 更新组件
  receiveComponent: function(
    internalInstance, nextElement, transaction, context
  ) {
    var prevElement = internalInstance._currentElement;

    if (nextElement === prevElement &&
        context === internalInstance._context
      ) {
      return;
    }

    var refsChanged = ReactRef.shouldUpdateRefs(
      prevElement,
      nextElement
    );

    if (refsChanged) {
      ReactRef.detachRefs(internalInstance, prevElement);
    }

    internalInstance.receiveComponent(nextElement, transaction, context);

    if (refsChanged &&
        internalInstance._currentElement &&
        internalInstance._currentElement.ref != null) {
      transaction.getReactMountReady().enqueue(attachRefs, internalInstance);
    }
  },

  // 刷新组件的变化
  performUpdateIfNecessary: function(
    internalInstance,
    transaction,
    updateBatchNumber
  ) {
    internalInstance.performUpdateIfNecessary(transaction);
  },
};
```

### Transaction

仅从 ReactUpdates 和 ReactReconciler 就能看出，React 渲染过程中很多的设计和处理都是包含在事务中，关于事务的说明请参考我之前整理的一篇文章 [React Transaction 机制](/react/react-transaction.html)。

### ReactUpdateQueue

事务处理过程中，对更新的处理都是批量进行，所以必须有队列保存这些更新，而且也需要将事务执行过程中传入的事件缓存下来，这就是 ReactUpdateQueue。还记得 ReactComponent 的 setState 和 forceUpdate 方法吗？实际都是调用 this.updater.enqueueXXX，这里的 updater 实际上对应的就是 ReactUpdateQueue，是在 ReactCompositeComponent 中初始化的。

```javascript
function enqueueUpdate(internalInstance) {
  ReactUpdates.enqueueUpdate(internalInstance);
}

var ReactUpdateQueue = {
  // 判断是否 mounted
  isMounted: function(publicInstance) {
    var internalInstance = ReactInstanceMap.get(publicInstance);
    if (internalInstance) {
      // During componentWillMount and render this will still be null but after
      // that will always render to something. At least for now. So we can use
      // this hack.
      return !!internalInstance._renderedComponent;
    } else {
      return false;
    }
  },

  // 回调入队，在所有挂起的更新处理完成后执行
  enqueueCallback: function(publicInstance, callback, callerName) {
    ReactUpdateQueue.validateCallback(callback, callerName);
    var internalInstance = getInternalInstanceReadyForUpdate(publicInstance);
    if (!internalInstance) {
      return null;
    }

    if (internalInstance._pendingCallbacks) {
      internalInstance._pendingCallbacks.push(callback);
    } else {
      internalInstance._pendingCallbacks = [callback];
    }
    enqueueUpdate(internalInstance);
  },

  enqueueCallbackInternal: function(internalInstance, callback) {
    if (internalInstance._pendingCallbacks) {
      internalInstance._pendingCallbacks.push(callback);
    } else {
      internalInstance._pendingCallbacks = [callback];
    }
    enqueueUpdate(internalInstance);
  },

  // 强制更新。当不在 DOM 事务时调用。
  enqueueForceUpdate: function(publicInstance) {
    var internalInstance = getInternalInstanceReadyForUpdate(
      publicInstance,
      'forceUpdate'
    );

    if (!internalInstance) {
      return;
    }

    internalInstance._pendingForceUpdate = true;
    enqueueUpdate(internalInstance);
  },

  // 替换所有 state。
  enqueueReplaceState: function(publicInstance, completeState) {
    var internalInstance = getInternalInstanceReadyForUpdate(
      publicInstance,
      'replaceState'
    );

    if (!internalInstance) {
      return;
    }

    internalInstance._pendingStateQueue = [completeState];
    internalInstance._pendingReplaceState = true;

    enqueueUpdate(internalInstance);
  },

  // Sets a subset of the state.
  enqueueSetState: function(publicInstance, partialState) {
    var internalInstance = getInternalInstanceReadyForUpdate(
      publicInstance,
      'setState'
    );

    if (!internalInstance) {
      return;
    }

    var queue =
      internalInstance._pendingStateQueue ||
      (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

    enqueueUpdate(internalInstance);
  },

  enqueueElementInternal: function(internalInstance, nextElement, nextContext) {
    internalInstance._pendingElement = nextElement;
    // TODO: introduce _pendingContext instead of setting it directly.
    internalInstance._context = nextContext;
    enqueueUpdate(internalInstance);
  },

  validateCallback: function(callback, callerName) {},
};
```

从 enqueueUpdate(internalInstance) 函数可以看出，入队的结果是交由 ReactUpdates.enqueueUpdate(internalInstance) 进行下一步处理。

### ReactCompositeComponent

组件间存在层层的嵌套关系，那么必然涉及到复合组件的渲染，下面介绍的 ReactCompositeComponent 就是处理这种场景。

ReactCompositeComponent 包含的逻辑太多，有兴趣的建议去读读源码，其中特别是对生命周期方法的处理比较有意思，能清晰的知道其执行流程或触发条件，有利于代码结构优化，提高性能。关于复合组件间生命周期的调用顺序，参考源码中的说明，已然非常详细。

```javascript
/**
 * ------------------ The Life-Cycle of a Composite Component ------------------
 *
 * - constructor: Initialization of state. The instance is now retained.
 *   - componentWillMount
 *   - render
 *   - [children's constructors]
 *     - [children's componentWillMount and render]
 *     - [children's componentDidMount]
 *     - componentDidMount
 *
 *       Update Phases:
 *       - componentWillReceiveProps (only called if parent updated)
 *       - shouldComponentUpdate
 *         - componentWillUpdate
 *           - render
 *           - [children's constructors or receive props phases]
 *         - componentDidUpdate
 *
 *     - componentWillUnmount
 *     - [children's componentWillUnmount]
 *   - [children destroyed]
 * - (destroyed): The instance is now blank, released by React and ready for GC.
 *
 * -----------------------------------------------------------------------------
 */
```

## DOM 渲染

本节介绍 DOM 元素的渲染实现。关于 React Native 的渲染，实现方法类似，不再介绍。

```javascript
var ReactDOM = {
  findDOMNode: findDOMNode,
  render: ReactMount.render,
  unmountComponentAtNode: ReactMount.unmountComponentAtNode,
  version: ReactVersion,

  unstable_batchedUpdates: ReactUpdates.batchedUpdates,
  unstable_renderSubtreeIntoContainer: renderSubtreeIntoContainer,
};
```

其中最关键的是 render 方法，来自 ReactMount 这个对象。

```javascript
var ReactMount = {

  TopLevelWrapper: TopLevelWrapper,

  // Take a component that's already mounted into the DOM and replace its props
  _updateRootComponent: function(
      prevComponent,
      nextElement,
      nextContext,
      container,
      callback) {...},

  // Render a new component into the DOM. Hooked by hooks!
  _renderNewRootComponent: function(
    nextElement,
    container,
    shouldReuseMarkup,
    context
  ) {
    ReactBrowserEventEmitter.ensureScrollValueMonitoring();
    var componentInstance = instantiateReactComponent(nextElement, false);

    ReactUpdates.batchedUpdates(
      batchedMountComponentIntoNode,
      componentInstance,
      container,
      shouldReuseMarkup,
      context
    );

    return componentInstance;
  },

  // Renders a React component into the DOM in the supplied `container`.
  renderSubtreeIntoContainer: function(parentComponent, nextElement, container, callback) {
    return ReactMount._renderSubtreeIntoContainer(
      parentComponent,
      nextElement,
      container,
      callback
    );
  },

  _renderSubtreeIntoContainer: function(parentComponent, nextElement, container, callback) {...},

  // Renders a React component into the DOM in the supplied `container`.
  render: function(nextElement, container, callback) {
    return ReactMount._renderSubtreeIntoContainer(null, nextElement, container, callback);
  },

  // Unmounts and destroys the React component rendered in the `container`.
  unmountComponentAtNode: function(container) {...},

  _mountImageIntoNode: function(
    markup,
    container,
    instance,
    shouldReuseMarkup,
    transaction
  ) {...},
};
```

ReactMount 代码太多，不过从关键部分也能看出其处理逻辑。

简单描述下新组件的渲染过程，\_renderNewRootComponent 在实例化时调用了 instantiateReactComponent 方法，实际上会调用 ReactCompositeComponent 处理复合组件，并返回一个实例。之后调用了 ReactUpdates.batchedUpdates 将渲染过程委托给 ReactUpdates 进行处理。在 ReactUpdates 中处理时会调用方法 batchedMountComponentIntoNode。

```javascript
function batchedMountComponentIntoNode(
  componentInstance,
  container,
  shouldReuseMarkup,
  context
) {
  var transaction = ReactUpdates.ReactReconcileTransaction.getPooled(
    /* useCreateElement */
    !shouldReuseMarkup && ReactDOMFeatureFlags.useCreateElement
  );
  transaction.perform(
    mountComponentIntoNode,
    null,
    componentInstance,
    container,
    transaction,
    shouldReuseMarkup,
    context
  );
  ReactUpdates.ReactReconcileTransaction.release(transaction);
}
```

batchedMountComponentIntoNode 会传入 transaction，调用其 perform 方法去执行 mountComponentIntoNode 函数。

```javascript
// Mounts this component and inserts it into the DOM.
function mountComponentIntoNode(
  wrapperInstance,
  container,
  transaction,
  shouldReuseMarkup,
  context
) {
  var markerName;
  if (ReactFeatureFlags.logTopLevelRenders) {
    var wrappedElement = wrapperInstance._currentElement.props.child;
    var type = wrappedElement.type;
    markerName = 'React mount: ' + (
      typeof type === 'string' ? type :
      type.displayName || type.name
    );
    console.time(markerName);
  }

  var markup = ReactReconciler.mountComponent(
    wrapperInstance,
    transaction,
    null,
    ReactDOMContainerInfo(wrapperInstance, container),
    context,
    0 /* parentDebugID */
  );

  if (markerName) {
    console.timeEnd(markerName);
  }

  wrapperInstance._renderedComponent._topLevelWrapper = wrapperInstance;
  ReactMount._mountImageIntoNode(
    markup,
    container,
    wrapperInstance,
    shouldReuseMarkup,
    transaction
  );
}
```

最终，mountComponentIntoNode 中执行 ReactMount.\_mountImageIntoNode 方法，将生成的 Virtual DOM 真正挂载到 DOM 节点上，整个渲染过程完成。


## Virtual DOM

Virtual DOM 是 ReactElement 树，并通过 ReactDOM 的 render 方法渲染到真实 DOM 上。

JSX 是语法糖，但并不是 HTML 的语法糖，其转化后的形式与 ReactElement 保持一致，如下：

```javascript
var Element = {
  type: 'div',
  props: {
    className: css,
    childlren: []
  }
};
```

而这正对应着内存中所描述的 Virtual DOM。Virtual DOM 类似于一种缓存机制，大大提升了与 DOM 节点交互的效率。
