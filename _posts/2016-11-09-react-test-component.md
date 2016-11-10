---
layout: post
title: 基于 React + Redux 的组件测试
keywords: react, redux, test, component, mocha, enzyme, chai, 组件, 测试
description: 基于 React + Redux 的组件测试
date: '2016-11-10 11:00:00 +0800'
categories: react, test
---

# 基于 React + Redux 的组件测试

本文以 web 项目为例，介绍 React + Redux 构建的应用如何对 component 进行测试，关于 action 或者 reducer 的测试方法请参考文章 [编写测试](http://cn.redux.js.org/docs/recipes/WritingTests.html)。

因为相关的文章有很多，也可查阅附属的参考内容。本文主要记录在编写和运行测试案例时遇到的问题，以及解决办法。

## 测试工具

- **mocha** 测试框架
- **enzyme** 测试工具库，Airbnb 出品，是 React 官方测试工具库的封装
- **chai** 断言库
- **sinon** 一个 mock 框架，可以对任何对象进行 mock，同时提供一些对 mock 对象的校验方法

## 常见问题

### 样式文件

在写 React 的 components 时可能会加上自己定义的一些 css 文件（或者是 less 和 sass 等），这在 mocha 运行测试时会报错，报无法解析 css 语法的错误。

解决办法： 安装库 `ignore-styles`，并在 mocha 命令后加上参数 ` --require ignore-styles` 即可。

### 使用 DOM 变量

在做 components 测试时还会遇到一个问题，比如在某些组件中使用了 DOM 的一些全局变量，比如 window，document 等，这些只有在浏览器中才会有，而 mocha 测试我们是在命令行中执行的，并没有浏览器的这些变量。

解决办法：测试用例中模拟 DOM 环境（即 window, document 和 navigator 对象），jsdom 库提供这项功能。

创建一个 setup.js 文件，内容如下：

```javascript
import { jsdom } from 'jsdom';

global.document = jsdom('');
global.window = document.defaultView;
Object.keys(document.defaultView).forEach(property => {
  if (typeof global[property] === 'undefined') {
    global[property] = document.defaultView[property];
  }
});

global.navigator = {
  userAgent: 'node.js'
};
```

然后，在 mocha 命令后加上参数 ` --require setup.js` 即可。

### 连接组件

如果你使用了 react-redux, 通常会用 connect() 将 redux store 的 state 或者 action 注入到组件中。

形式为：

```javascript
import { connect } from 'react-redux'

class App extends Component { /* ... */ }

export default connect(mapStateToProps)(App)
```

在单元测试中，一般会这样导入 App 组件

```javascript
import App from './App'
```

但是，当这样导入时，实际上持有的是 connect() 返回的包装过组件，而不是 App 组件本身。如果想测试它和 Redux 间的互动，好消息是可以使用一个专为单元测试创建的 store， 将它包装在<Provider> 中。但有时我们仅仅是想测试组件的渲染，并不想要这么一个 Redux store。

这时，可以采用如下写法。

```javascript
import { connect } from 'react-redux'

// 命名导出未连接的组件 (测试用)
export class App extends Component { /* ... */ }

// 默认导出已连接的组件 (app 用)
export default connect(mapDispatchToProps)(App)
```

鉴于默认导出的依旧是包装过的组件，上面的导入语句会和之前一样工作，不需要更改应用中的代码。不过，可以这样在测试文件中导入没有包装的 App 组件：

```javascript
// 注意花括号：抓取命名导出，而不是默认导出
import { App } from './App'
```

在 app 中，仍然正常地导入：

```javascript
import App from './App'
```

只在测试中使用命名导出。


## 参考

[前端单元测试探索](https://segmentfault.com/a/1190000006933557)

[测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

[Enzyme](http://airbnb.io/enzyme/)

[使用 Mocha + Chai + Sinon 测试 React + Redux 的 web 应用](http://zhaozhiming.github.io/blog/2015/12/19/use-mocha-and-chai-and-sinon-to-test-react-and-redux-webapp/)

[React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)
