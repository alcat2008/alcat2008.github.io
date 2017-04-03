---
layout: post
title: 基于 React + Redux 的组件测试
keywords: react, redux, test, component, mocha, enzyme, chai, 组件, 测试
description: 基于 React + Redux 的组件测试
date: '2016-11-10 11:00:00 +0800'
tags: [react, redux, test]
categories: react
---

# 基于 React + Redux 的组件测试

基于 React + Redux 构建的 web 应用如何进行自动化测试？关于 action 或者 reducer 的测试方法请参考文章 [编写测试](http://cn.redux.js.org/docs/recipes/WritingTests.html)，下文主要介绍对 React component 的测试方法，并记录测试过程中遇到的问题及解决办法。

## TDD (Test-driven development)

测试驱动开发（TDD），其基本思路是 **通过测试来推动整个开发的进行**。

- 单元测试的首要目的不是为了能够编写出大覆盖率的全部通过的测试代码，而是需要从使用者(调用者)的角度出发，尝试函数逻辑的各种可能性，进而辅助性增强代码质量
- 测试是手段而不是目的。测试的主要目的不是证明代码正确，而是帮助发现错误，包括低级的错误
- 测试要快。快速运行、快速编写
- 测试代码保持简洁
- 不会忽略失败的测试。一旦团队开始接受1个测试的构建失败，那么他们渐渐地适应2、3、4或者更多的失败。在这种情况下，测试集就不再起作用

IMPORTANT

- 一定不能误解了TDD的核心目的！
- 测试不是为了覆盖率和正确率
- 而是作为实例，告诉开发人员要编写什么代码
- 红灯（代码还不完善，测试挂）-> 绿灯（编写代码，测试通过）-> 重构（优化代码并保证测试通过）

大致过程

1. 需求分析，思考实现。考虑如何“使用”产品代码，是一个实例方法还是一个类方法，是从构造函数传参还是从方法调用传参，方法的命名，返回值等。这时其实就是在做设计，而且设计以代码来体现。此时测试为红
2. 实现代码让测试为绿
3. 重构，然后重复测试
4. 最终符合所有要求：
  - 每个概念都被清晰的表达
  - Not Repeat Self
  - 没有多余的东西
  - 通过测试

## 单元测试

### 什么是单元测试？

> In computer programming, unit testing is a software testing method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine whether they are fit for use.

> 在计算机编程中，单元测试（英语：Unit Testing）又称为模块测试, 是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。

### 收益

单元测试的目标是隔离程序部件并证明这些单个部件是正确的。一个单元测试提供了代码片断需要满足的严密的书面规约。

- 适应变更
  单元测试允许程序员在未来重构代码，并且确保模块依然工作正确。

- 简化集成
  单元测试消除程序单元的不可靠，采用自底向上的测试路径。通过先测试程序部件再测试部件组装，使集成测试变得更加简单。

- 文档记录
  单元测试提供了系统的一种文档记录。借助于查看单元测试提供的功能和单元测试中如何使用程序单元，开发人员可以直观的理解程序单元的基础API。

- 表达设计
  在测试驱动开发的软件实践中，单元测试可以替换正式的设计。每一个单元测试案例均可以视为一项类、方法和待观察行为等设计元素。

### 原则

- 测试代码时，只考虑测试，不考虑内部实现
- 数据尽量模拟现实，越靠近现实越好
- 充分考虑数据的边界条件
- 对重点、复杂、核心代码，重点测试
- 利用 AOP (beforeEach、afterEach)，减少测试代码数量，避免无用功能
- 测试、功能开发相结合，有利于设计和代码重构


## 测试工具

- **mocha** 测试框架
- **enzyme** 测试工具库，Airbnb 出品，是 React 官方测试工具库的封装
- **chai** 单元测试的验证框架
- **sinon** 一个 mock 框架，可以对任何对象进行 mock，同时提供一些对 mock 对象的校验方法

## React 测试

React 测试必须使用官方的[测试工具库](https://facebook.github.io/react/docs/test-utils.html)，但是它用起来不够方便，所以有人做了封装，推出了一些第三方库，其中 Airbnb 公司的 Enzyme 最容易上手。

### 测试对象

一个 React 组件有两种存在形式：**虚拟 DOM 对象**（即 React.Component 的实例）和 **真实 DOM 节点**。

- Shallow Rendering：测试虚拟 DOM 的方法
  Shallow Rendering （浅渲染）指的是，将一个组件渲染成虚拟 DOM 对象，但是只渲染第一层，不渲染所有子组件，所以处理速度非常快。它不需要DOM环境，因为根本没有加载进 DOM。

- DOM Rendering: 测试真实 DOM 的方法
  将组件渲染成真实的 DOM 节点，再进行测试。该方法要求存在一个真实的 DOM 环境，否则会报错。

### enzyme

enzyme 是官方测试工具库的封装，它模拟了 jQuery 的 API，非常直观，易于使用和学习。它提供三种测试方法。

- shallow
  [shallow](https://github.com/airbnb/enzyme/blob/master/docs/api/shallow.md) 方法就是官方的 shallow rendering 的封装。

- render
  [render](https://github.com/airbnb/enzyme/blob/master/docs/api/render.md) 方法将 React 组件渲染成静态的 HTML 字符串，然后分析这段 HTML 代码的结构，返回一个对象。

- mount
  [mount](https://github.com/airbnb/enzyme/blob/master/docs/api/mount.md) 方法用于将 React 组件加载为真实 DOM 节点。


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

[测试 Redux 应用中使用 window.fetch 的 API 请求](http://frantic1048.logdown.com/posts/517113-test-redux-application-windowfetch-api-request)
