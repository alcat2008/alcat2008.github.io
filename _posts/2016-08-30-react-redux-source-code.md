---
layout: post
title: react-redux 源码分析
keywords: react-redux, analysis, 分析, 源码
description: react-redux 源码分析解读
date: '2016-08-30 08:00:00 +0800'
tags: [React, Redux]
categories: architecture
---


# react-redux 源码分析

react-redux 是 Redux 官方提供的 React 绑定库，具有高效且灵活的特性。

关于 react-redux 的使用方法，请参考文章 [UsageWithReact](http://redux.js.org/docs/basics/UsageWithReact.html)，或者中文版 [搭配 React](http://cn.redux.js.org/docs/basics/UsageWithReact.html)。

本文试图对 react-redux 源码进行解读，因水平有限，不当之处，敬请指出，不胜感激！源码中会省略一些无关代码，并用 `---` 进行注解。

[react-redux 官方 github](https://github.com/reactjs/react-redux) 的源码目录结构如下，其核心是 Provider 和 connect 两个 API。

```javascript
├── components/
│     ├── connect.js
│     ├── Provider.js
├── utils/
│     ├── shallowEqual.js // 对象判等
│     ├── storeShap.js // store 的结构，须包含 subscribe、dispatch、getState 属性
│     ├── warning.js
│     ├── wrapActionCreators.js // bindActionCreators 包装
├── index.js // entry file
```

## Provider

Provider 提供了全局的 context，组件只有嵌套在 `<Provider>` 中才能使用 connect() 方法。`<Provider store>` 使组件层级中的 connect() 方法都能够获得 store。

```javascript
import { Component, PropTypes, Children } from 'react'
import storeShape from '../utils/storeShape'

export default class Provider extends Component {
  // --- 通过 context 向下传递 store，可以被子组件引用
  getChildContext() {
    return { store: this.store }
  }

  constructor(props, context) {
    super(props, context)
    // --- 获取传入的 store
    this.store = props.store
  }

  render() {
    return Children.only(this.props.children)
  }
}

Provider.propTypes = {
  store: storeShape.isRequired,
  children: PropTypes.element.isRequired
}
// --- 验证 store 的格式
Provider.childContextTypes = {
  store: storeShape.isRequired
}
```

## connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {})

connect 连接 React 组件与 Redux store。连接操作不会改变原来的组件类，而是返回一个新的与 store 连接的组件类。

```javascript
import { Component, createElement } from 'react'
import storeShape from '../utils/storeShape'
import shallowEqual from '../utils/shallowEqual'
import wrapActionCreators from '../utils/wrapActionCreators'
import warning from '../utils/warning'
import isPlainObject from 'lodash/isPlainObject'
import hoistStatics from 'hoist-non-react-statics'

// --- 定义默认的参数、方法
const defaultMapStateToProps = state => ({})
const defaultMapDispatchToProps = dispatch => ({ dispatch })
const defaultMergeProps = (stateProps, dispatchProps, parentProps) => ({
  ...parentProps,
  ...stateProps,
  ...dispatchProps
})

// --- 用于设置 displayName
function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component'
}

// --- 安全执行
let errorObject = { value: null }
function tryCatch(fn, ctx) {
  try {
    return fn.apply(ctx)
  } catch (e) {
    errorObject.value = e
    return errorObject
  }
}

export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
  // --- 根据是否定义 mapStateToProps 加 hack 标志
  // --- 注意：Boolean({}), Boolean(state => ({})) 的结果都是 true
  const shouldSubscribe = Boolean(mapStateToProps)
  const mapState = mapStateToProps || defaultMapStateToProps

  // --- 构造最终的 mapDispatch
  let mapDispatch
  if (typeof mapDispatchToProps === 'function') {
    mapDispatch = mapDispatchToProps
  } else if (!mapDispatchToProps) {
    mapDispatch = defaultMapDispatchToProps
  } else {
    // --- 会自动用 bindActionCreators 包装
    mapDispatch = wrapActionCreators(mapDispatchToProps)
  }

  const finalMergeProps = mergeProps || defaultMergeProps
  const { pure = true, withRef = false } = options
  const checkMergedEquals = pure && finalMergeProps !== defaultMergeProps

  return function wrapWithConnect(WrappedComponent) {
    // --- 被 wrap 组件的 displayName
    const connectDisplayName = `Connect(${getDisplayName(WrappedComponent)})`

    // --- 计算 props
    function computeMergedProps(stateProps, dispatchProps, parentProps) {
      const mergedProps = finalMergeProps(stateProps, dispatchProps, parentProps)
      return mergedProps
    }

    class Connect extends Component {
      shouldComponentUpdate() {
        return !pure || this.haveOwnPropsChanged || this.hasStoreStateChanged
      }

      constructor(props, context) {
        super(props, context)
        this.version = version
        this.store = props.store || context.store

        const storeState = this.store.getState()
        this.state = { storeState }

        // --- 初始化/重置 实例属性
        this.clearCache()
      }

      // --- computeStateProps、configureFinalMapState 计算 state props
      computeStateProps(store, props) {
        if (!this.finalMapStateToProps) {
          return this.configureFinalMapState(store, props)
        }

        const state = store.getState()
        const stateProps = this.doStatePropsDependOnOwnProps ?
          this.finalMapStateToProps(state, props) :
          this.finalMapStateToProps(state)

        return stateProps
      }

      configureFinalMapState(store, props) {
        const mappedState = mapState(store.getState(), props)
        const isFactory = typeof mappedState === 'function'

        this.finalMapStateToProps = isFactory ? mappedState : mapState
        this.doStatePropsDependOnOwnProps = this.finalMapStateToProps.length !== 1

        if (isFactory) {
          return this.computeStateProps(store, props)
        }

        return mappedState
      }

      // --- computeDispatchProps、configureFinalMapDispatch 计算 dispatch props
      computeDispatchProps(store, props) {
        if (!this.finalMapDispatchToProps) {
          return this.configureFinalMapDispatch(store, props)
        }

        const { dispatch } = store
        const dispatchProps = this.doDispatchPropsDependOnOwnProps ?
          this.finalMapDispatchToProps(dispatch, props) :
          this.finalMapDispatchToProps(dispatch)

        return dispatchProps
      }

      configureFinalMapDispatch(store, props) {
        const mappedDispatch = mapDispatch(store.dispatch, props)
        const isFactory = typeof mappedDispatch === 'function'

        this.finalMapDispatchToProps = isFactory ? mappedDispatch : mapDispatch
        this.doDispatchPropsDependOnOwnProps = this.finalMapDispatchToProps.length !== 1

        if (isFactory) {
          return this.computeDispatchProps(store, props)
        }

        return mappedDispatch
      }

      // --- 对比 connect 中映射 store 的 state 变化
      updateStatePropsIfNeeded() {
        const nextStateProps = this.computeStateProps(this.store, this.props)
        if (this.stateProps && shallowEqual(nextStateProps, this.stateProps)) {
          return false
        }

        this.stateProps = nextStateProps
        return true
      }

      // --- 对比 connect 中映射 store 的 actions 变化
      updateDispatchPropsIfNeeded() {
        const nextDispatchProps = this.computeDispatchProps(this.store, this.props)
        if (this.dispatchProps && shallowEqual(nextDispatchProps, this.dispatchProps)) {
          return false
        }

        this.dispatchProps = nextDispatchProps
        return true
      }

      // --- 对比 props 变化
      updateMergedPropsIfNeeded() {
        const nextMergedProps = computeMergedProps(this.stateProps, this.dispatchProps, this.props)
        if (this.mergedProps && checkMergedEquals && shallowEqual(nextMergedProps, this.mergedProps)) {
          return false
        }

        this.mergedProps = nextMergedProps
        return true
      }

      // --- 判断是否注册
      isSubscribed() {
        return typeof this.unsubscribe === 'function'
      }

      // --- 注册监听器
      trySubscribe() {
        if (shouldSubscribe && !this.unsubscribe) {
          this.unsubscribe = this.store.subscribe(this.handleChange.bind(this))
          this.handleChange()
        }
      }

      // --- 注销监听器
      tryUnsubscribe() {
        if (this.unsubscribe) {
          this.unsubscribe()
          this.unsubscribe = null
        }
      }

      componentDidMount() {
        this.trySubscribe()
      }

      componentWillReceiveProps(nextProps) {
        if (!pure || !shallowEqual(nextProps, this.props)) {
          this.haveOwnPropsChanged = true
        }
      }

      componentWillUnmount() {
        this.tryUnsubscribe()
        this.clearCache()
      }

      clearCache() {
        this.dispatchProps = null
        this.stateProps = null
        this.mergedProps = null
        this.haveOwnPropsChanged = true
        this.hasStoreStateChanged = true
        this.haveStatePropsBeenPrecalculated = false
        this.statePropsPrecalculationError = null
        this.renderedElement = null
        this.finalMapDispatchToProps = null
        this.finalMapStateToProps = null
      }

      // --- 响应 store 的变化
      handleChange() {
        if (!this.unsubscribe) {
          return
        }

        const storeState = this.store.getState()
        const prevStoreState = this.state.storeState
        if (pure && prevStoreState === storeState) {
          return
        }

        if (pure && !this.doStatePropsDependOnOwnProps) {
          // --- 根据 state 变化，判断是否更新当前组件
          const haveStatePropsChanged = tryCatch(this.updateStatePropsIfNeeded, this)
          if (!haveStatePropsChanged) {
            return
          }
          if (haveStatePropsChanged === errorObject) {
            this.statePropsPrecalculationError = errorObject.value
          }
          this.haveStatePropsBeenPrecalculated = true
        }

        this.hasStoreStateChanged = true
        this.setState({ storeState })
      }

      // --- 获取组件实例
      getWrappedInstance() {
        invariant(withRef,
          `To access the wrapped instance, you need to specify ` +
          `{ withRef: true } as the fourth argument of the connect() call.`
        )

        return this.refs.wrappedInstance
      }

      render() {
        const {
          haveOwnPropsChanged,
          hasStoreStateChanged,
          haveStatePropsBeenPrecalculated,
          statePropsPrecalculationError,
          renderedElement
        } = this

        this.haveOwnPropsChanged = false
        this.hasStoreStateChanged = false
        this.haveStatePropsBeenPrecalculated = false
        this.statePropsPrecalculationError = null

        if (statePropsPrecalculationError) {
          throw statePropsPrecalculationError
        }

        let shouldUpdateStateProps = true
        let shouldUpdateDispatchProps = true
        if (pure && renderedElement) {
          shouldUpdateStateProps = hasStoreStateChanged || (
            haveOwnPropsChanged && this.doStatePropsDependOnOwnProps
          )
          shouldUpdateDispatchProps =
            haveOwnPropsChanged && this.doDispatchPropsDependOnOwnProps
        }

        let haveStatePropsChanged = false
        let haveDispatchPropsChanged = false
        if (haveStatePropsBeenPrecalculated) {
          haveStatePropsChanged = true
        } else if (shouldUpdateStateProps) {
          haveStatePropsChanged = this.updateStatePropsIfNeeded()
        }
        if (shouldUpdateDispatchProps) {
          haveDispatchPropsChanged = this.updateDispatchPropsIfNeeded()
        }

        let haveMergedPropsChanged = true
        if (
          haveStatePropsChanged ||
          haveDispatchPropsChanged ||
          haveOwnPropsChanged
        ) {
          haveMergedPropsChanged = this.updateMergedPropsIfNeeded()
        } else {
          haveMergedPropsChanged = false
        }

        if (!haveMergedPropsChanged && renderedElement) {
          return renderedElement
        }

        if (withRef) {
          this.renderedElement = createElement(WrappedComponent, {
            ...this.mergedProps,
            ref: 'wrappedInstance'
          })
        } else {
          this.renderedElement = createElement(WrappedComponent,
            this.mergedProps
          )
        }

        return this.renderedElement
      }
    }

    Connect.displayName = connectDisplayName
    Connect.WrappedComponent = WrappedComponent

    // --- 从 context 获取 store
    Connect.contextTypes = {
      store: storeShape
    }
    Connect.propTypes = {
      store: storeShape
    }

    // --- 复制被 wrap 组件的 non-react specific statics
    return hoistStatics(Connect, WrappedComponent)
  }
}
```

## shallowEqual(objA, objB)

在 connect 的监听器事件中，使用 shallowEqual 来对比 state 是否有变化，所以理解其原理，有助于我们优化 state 结构，进而提升渲染性能。

```javascript
export default function shallowEqual(objA, objB) {
  // --- 引用比较，所以若在 reducer 中返回相同的引用，将不会触发渲染
  if (objA === objB) {
    return true
  }

  // --- 遍历对象属性并对比
  const keysA = Object.keys(objA)
  const keysB = Object.keys(objB)

  if (keysA.length !== keysB.length) {
    return false
  }

  // Test for A's keys different from B.
  const hasOwn = Object.prototype.hasOwnProperty
  for (let i = 0; i < keysA.length; i++) {
    if (!hasOwn.call(objB, keysA[i]) ||
        objA[keysA[i]] !== objB[keysA[i]]) {
      return false
    }
  }

  return true
}
```

## 总结

react-redux 最大的作用就是将 store 绑定到组件上，并在 state 更新时有选择的重新渲染。
