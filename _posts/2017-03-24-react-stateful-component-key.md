---
layout: post
title: React 有状态组件及 key 属性的应用
keywords: react, 有状态, stateful, key, react-router, reload, refresh
description: React 有状态组件及 key 属性的应用，提供解决 react-router 中 route 组件刷新的新思路
date: '2017-03-24 08:00:00 +0800'
categories: react
---

# React 有状态组件及 key 属性的应用

## 有状态组件

在构建 React 组件时，一个较为常见的场景是需要通过 props 计算得到某些渲染数据，实现方式无外乎以下两种：

（一） constructor 中对 props 进行处理，用 state 承载最终的渲染数据。
（二） 直接在 render 中计算并使用计算结果进行渲染。

两种方式各有优劣：

- 方法一在实现时往往需要配合 componentWillReceiveProps 一起使用，为何？一旦用于计算的 props 发生变化，那么就需要对 state 同步更新；
- 方法二每次 render 都需要计算一次，无法忽视其对性能的影响。

下面顺着这个思路思考下去，究竟如何构建一个复杂的有状态组件？网上已经有太多的最佳实践，都各有千秋，这里不再赘述。在我看来，组件的设计其实就是一种权衡，其首要考虑是模块分解的合理性、代码的可维护性。

这里强调一点，在React 组件中，一旦需要在 componentWillReceiveProps 中进行某些 setState 的操作，那么首先应当考虑 props 或者 state 是否需要重新设计。因为在设计良好的 React 组件中，props 与 state 所包含的信息应是正交的关系。

## key 属性

如果组件是复杂的有状态组件，这种情况无法避免，而你又希望在每次映射（比如该组件对应同样的一个路由，唯一的区别就是参数）该组件时能重建该组件，可以考虑尝试下 key 属性。

为何 key 属性能解决这个问题？看看其实现，这才是最靠谱的。ReactChildReconciler 是 React 操作 children 的具体实现，包括初始化、更新及销毁。

主要关注 updateChildren 的逻辑, key 属性变化时的逻辑处理源码摘抄如下：

```javascript
updateChildren: function(prevChildren, nextChildren, ...) {
    ...
    for (name in nextChildren) {
        ...
        // The child must be instantiated before it's mounted.
        var nextChildInstance = instantiateReactComponent(nextElement, true);
        nextChildren[name] = nextChildInstance;
        // Creating mount image now ensures refs are resolved in right order
        // (see https://github.com/facebook/react/pull/7101 for explanation).
        var nextChildMountImage = ReactReconciler.mountComponent(
          nextChildInstance,
          transaction,
          hostParent,
          hostContainerInfo,
          context,
          selfDebugID,
        );
        mountImages.push(nextChildMountImage);
        ...
    }
    // Unmount children that are no longer present.
    for (name in prevChildren) {
        ...
        prevChild = prevChildren[name];
        removedNodes[name] = ReactReconciler.getHostNode(prevChild);
        ReactReconciler.unmountComponent(
          prevChild,
          false /* safely */,
          false /* skipLifecycle */,
        );
        ...
    }
  },
```

所以当 key 变化时会删除 prevChild， 并新建 nextChild。所以通过 key 属性，能在合适的时候触发组件的销毁与重建。


## react-router

回到具体的应用场景，使用 react-router 时， route 组件通常都是较为复杂的有状态组件，如何在路由页面间自由跳转并自动触发组件重建是个较为棘手的问题，因为如果是映射的同一组件，只会 re-render。有了 key，我们就可以在 route 组件上根据 params 构建 key，那么上述问题就能解决了。

同时附上传 props 给 route 组件的方法，供参考。

```javascript
class App extends Component {
  static createElement = (Component, ownProps) => {
    const { userId } = ownProps.params;
    switch (Component) {
      case UserDashboard:
        return <Component key={userId} {...ownProps} />;
      default:
        return <Component {...ownProps} />;
    }
  };
  render() {
    return (
      <Provider store={store}>
        <Router createElement={App.createElement} history={syncHistoryWithStore(hashHistory, store)}>
          <Route path="/" component={Home}>
            <IndexRoute component={Index}/>
            <Route path="users/:userId" component={UserDashboard}/>
          </Route>
        </Router>
      </Provider>
    )
  }
}
```

## 结语

虽然 key 属性有这么大能量，但还是不能滥用，需要权衡，毕竟重建是一个比较消耗资源的行为。本文只不过是提供一种解决问题的思路，完美的解决方案还是在设计时对逻辑进行隔离，从源头上避免一些反模式，做到真正的组件化。


## 参考资料

[how to reload a route?](https://github.com/ReactTraining/react-router/issues/1982)
[React 实践心得：key 属性的原理和用法](http://taobaofed.org/blog/2016/08/24/react-key/)
