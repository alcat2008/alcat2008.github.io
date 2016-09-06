---
layout: post
title: Redux 源码分析
date: '2016-08-23 08:00:00 +0800'
categories: architecture
---

# Redux 源码分析

Redux 的设计理念很简单，也很优雅，具体可参见我的学习笔记 [理解 Redux](/architecture/redux.html)。

本文试图对 Redux 源码进行解读，因水平有限，不当之处，敬请指出，不胜感激！源码中会省略一些无关代码，并用 `---` 进行注解。

可以在 [Redux 官方 github](https://github.com/reactjs/redux/tree/master/src) 目录中直接查看到源码，结构很精简，除去 index.js 文件和 utils 目录（实际只有 warning.js 文件）外只包含 5 个文件，分别对应 5 个 API。

```javascript
├── utils/
│     ├── warning.js
├── applyMiddleware.js
├── bindActionCreators.js
├── combineReducers.js
├── compose.js
├── createStore.js
├── index.js // entry file
```

## createStore(reducer, preloadedState, enhancer)

createStore 是整个 Redux 的中枢，负责创建 store。

```javascript

// --- 默认的 INIT action
export var ActionTypes = {
  INIT: '@@redux/INIT'
}

/**
 * Creates a Redux store that holds the state tree.
 * The only way to change the data in the store is to call `dispatch()` on it.
 *
 * There should only be a single store in your app. To specify how different
 * parts of the state tree respond to actions, you may combine several reducers
 * into a single reducer function by using `combineReducers`.
 *
 * @param {Function} reducer A function that returns the next state tree, given
 * the current state tree and the action to handle.
 * --- reducer 是 Function，用于构建 state 树
 *
 * @param {any} [preloadedState] The initial state. You may optionally specify it
 * to hydrate the state from the server in universal apps, or to restore a
 * previously serialized user session.
 * If you use `combineReducers` to produce the root reducer function, this must be
 * an object with the same shape as `combineReducers` keys.
 * --- preloadedState 初始化 state 树
 *
 * @param {Function} enhancer The store enhancer. You may optionally specify it
 * to enhance the store with third-party capabilities such as middleware,
 * time travel, persistence, etc. The only store enhancer that ships with Redux
 * is `applyMiddleware()`.
 * --- enhancer 相当于 AOP 插件
 *
 * @returns {Store} A Redux store that lets you read the state, dispatch actions
 * and subscribe to changes.
 * --- 最终返回 Store，能获取 state、分发action、订阅变化
 */
export default function createStore(reducer, preloadedState, enhancer) {
  ... // --- 校验参数，及执行 enhancer

  var currentReducer = reducer
  var currentState = preloadedState // --- state 树
  var currentListeners = [] // --- 注册的监听器列表，实时处理 dispatch 事件
  var nextListeners = currentListeners // --- 注册的监听器列表，实时接收 subscribe 事件
  var isDispatching = false // --- 如果 reducer 正在执行，会抛出异常

  // --- 将 nextListeners 做为 currentListeners 的副本
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  // --- 返回 state
  function getState() {
    ... // --- 校验

    return currentState
  }

  // --- 注册监听器
  function subscribe(listener) {
    ... // --- 校验

    var isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    // --- 返回了用于注销监听器的方法
    return function unsubscribe() {
      ... // --- 校验

      isSubscribed = false

      ensureCanMutateNextListeners()
      var index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  // --- 分发 action，修改 state 的唯一方式
  function dispatch(action) {
    ... // --- 校验

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action) // --- 执行 reducer，state 被更新
    } finally {
      isDispatching = false
    }

    // --- 调用监听器方法
    var listeners = currentListeners = nextListeners
    for (var i = 0; i < listeners.length; i++) {
      listeners[i]()
    }

    // --- 返回结果还是 action
    return action
  }

  // --- 替换 reducer，用于热替换、按需加载等场景
  function replaceReducer(nextReducer) {
    ... // --- 校验

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.INIT })
  }

  // --- 预留给 可观察/响应式库 的接口
  // --- 请参考 https://github.com/zenparsing/es-observable
  function observable() {
    // --- 略
  }

  // --- 创建时初始化应用状态
  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

## combineReducers(reducers)

用于合并 reducer

```javascript
/**
 * Turns an object whose values are different reducer functions, into a single
 * reducer function. It will call every child reducer, and gather their results
 * into a single state object, whose keys correspond to the keys of the passed
 * reducer functions.
 *
 * @param {Object} reducers An object whose values correspond to different
 * reducer functions that need to be combined into one. One handy way to obtain
 * it is to use ES6 `import * as reducers` syntax. The reducers may never return
 * undefined for any action. Instead, they should return their initial state
 * if the state passed to them was undefined, and the current state for any
 * unrecognized action.
 *
 * @returns {Function} A reducer function that invokes every reducer inside the
 * passed object, and builds a state object with the same shape.
 */
export default function combineReducers(reducers) {
  // --- 过滤 reducer
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  var finalReducerKeys = Object.keys(finalReducers)

  // --- 返回 combination 函数
  return function combination(state = {}, action) {
    // --- 校验

    // --- 根据 reducer key 及执行结果构造 state 树
    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action) // --- 执行 reducer
      // --- 校验
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

## compose(...funcs)

compose 是 pure function，主要作用是组合传入的函数。

```javascript
/**
 * Composes single-argument functions from right to left. The rightmost
 * function can take multiple arguments as it provides the signature for
 * the resulting composite function.
 *
 * @param {...Function} funcs The functions to compose.
 * @returns {Function} A function obtained by composing the argument functions
 * from right to left. For example, compose(f, g, h) is identical to doing
 * (...args) => f(g(h(...args))).
 */

export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  // --- 巧妙的 reduce 应用
  return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}
```

## applyMiddleware(...middlewares)

applyMiddleware 传入 middleware 链，并返回应用这些 middleware 的 store enhancer。

```javascript
import compose from './compose'

/**
 * Creates a store enhancer that applies middleware to the dispatch method
 * of the Redux store. This is handy for a variety of tasks, such as expressing
 * asynchronous actions in a concise manner, or logging every action payload.
 *
 * See `redux-thunk` package as an example of the Redux middleware.
 *
 * Because middleware is potentially asynchronous, this should be the first
 * store enhancer in the composition chain.
 *
 * Note that each middleware will be given the `dispatch` and `getState` functions
 * as named arguments.
 *
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  // --- 传入 createStore，返回增强版的 store
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    // --- middleware 中的参数
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }

    // --- 给 middleware 传参
    chain = middlewares.map(middleware => middleware(middlewareAPI))

    // --- 组合 chain，传入原始的 dispatch
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store, // --- 保留 subscribe, getState, replaceReducer, [$$observable]: observable
      dispatch // --- 用新 dispatch 覆盖，之后调用 dispatch 就会触发 chain 内的 middleware 链式执行
    }
  }
}
```

## bindActionCreators(actionCreators, dispatch)

bindActionCreators 把 action creators 转成拥有同名 keys 的对象，使用 dispatch 把每个 action creator 包装起来，这样可以直接调用它们。


```javascript
// --- 用 dispatch 把每个 action creator 包装起来
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

/**
 * Turns an object whose values are action creators, into an object with the
 * same keys, but with every function wrapped into a `dispatch` call so they
 * may be invoked directly. This is just a convenience method, as you can call
 * `store.dispatch(MyActionCreators.doSomething())` yourself just fine.
 *
 * For convenience, you can also pass a single function as the first argument,
 * and get a function in return.
 *
 * @param {Function|Object} actionCreators An object whose values are action
 * creator functions. One handy way to obtain it is to use ES6 `import * as`
 * syntax. You may also pass a single function.
 *
 * @param {Function} dispatch The `dispatch` function available on your Redux
 * store.
 *
 * @returns {Function|Object} The object mimicking the original object, but with
 * every action creator wrapped into the `dispatch` call. If you passed a
 * function as `actionCreators`, the return value will also be a single
 * function.
 */
export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  // --- 校验

  // --- 遍历 action creators，分别调用 bindActionCreator 进行包装
  var keys = Object.keys(actionCreators)
  var boundActionCreators = {}
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i]
    var actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```

## 总结

Redux 的源码确实非常优雅。本文只是对关键步骤进行注解，希望能对初学者有些帮助。若有不当之处，请及时联系我，也欢迎各位一起讨论。
