---
layout: post
title: 前端测试框架概览
keywords: 前端, front-end, 测试
description: 前端测试框架概览
date: '2017-02-17 11:00:00 +0800'
categories: test
---

# 前端测试框架概览


## 测试工具

### Mocha

Mocha 是一个 JavaScript 的测试框架，类似于 Java 中的 Junit。

[测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)


### Chai

Chai 是一个单元测试的验证框架，它有 3 种不同形式的校验：expect、should 和 assert。expect 和 should 的方式让写出来的测试代码更像自然语言，让业务人员也可以看懂，而 assert 方式是传统单元测试断言的方式，如果以前习惯写 Java 的单元测试会对这种方式比较熟悉。


### Sinon

Sinon 是一个 mock 框架，可以对任何对象进行 mock，更重要的是它提供了一些对 mock 对象的校验方法。


### Enzyme

Enzyme 是 React 官方测试工具库的封装，它模拟了 jQuery 的 API，非常直观，易于使用和学习。

[Enzyme](http://airbnb.io/enzyme/)


### Karma

Karma 是一个基于 Node.js 的 JavaScript 测试执行过程管理工具（Test Runner）。该工具可用于测试所有主流 Web 浏览器，也可集成到 CI（Continuous integration） 工具，也可和其他代码编辑器一起使用。这个测试工具的一个强大特性就是，它可以监控 (Watch) 文件的变化，然后自行执行，通过 console.log 显示测试结果。

[karma 测试框架的前世今生](http://taobaofed.org/blog/2016/01/08/karma-origin/)


### Jasmine

Jasmine 是一款 JavaScript 测试框架，它不依赖于其他任何 JavaScript 组件。它有干净清晰的语法，让您可以很简单的写出测试代码。

[JavaScript 单元测试框架：Jasmine 初探](https://www.ibm.com/developerworks/cn/web/1404_changwz_jasmine/)


### Selenium

Selenium 是专门为 Web 应用程序编写的一个验收测试工具，它直接运行在浏览器中。Selenium 测试通常会调起一个可见的界面，但也可以通过设置，让它以 PhantomJS 的形式进行无界面的测试。

Selenium 是一个完整的测试工具，它是一个比较成熟的古老测试工具之一，它比较适合用来做高级测试，比如 e2e 测试。

它的核心思想就是代理注入，Selenium 通过代理打开浏览器 URL， 有一个后台的 server 在监听，当有请求时，会往浏览器中注入脚本，之后就通过这个脚本来跟 server 端通讯。

开发者可以使用 Selenium 客户端 API 来编写单测， 比如跳转 URL或者单击按钮


### PhantomJS

PhantomJS 是一个基于 webkit 的服务器端 JavaScript API，即相当于在内存中跑了个无界面的 webkit 内核的浏览器。通过它我们可以模拟页面加载，并获取到页面上的 DOM 元素，进行一系列的操作，以此来模拟UI测试。但缺点是无法实时看见页面上的情况（不过可以截图）。

其支持各种Web标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG。对于web测试、界面、网络捕获、页面自动化访问等等方面可以说是信手拈来。


### CasperJS

CasperJS 是一个开源的导航脚本和测试工具，使用 JavaScript 基于 PhantomJS 编写，用于测试 Web 应用功能.


### Istanbul

JavaScript 程序的代码覆盖率工具

[代码覆盖率工具 Istanbul 入门教程](http://www.ruanyifeng.com/blog/2015/06/istanbul.html)


### Jest

Facebook 发布的一个开源的、基于 Jasmine 框架的 JavaScript 单元测试工具。


### Nightwatch

Nightwatch 是一个易于使用的，基于 Node.js 平台的浏览器自动化测试解决方案。它使用强大的 Selenium WebDriver API 来在 DOM 元素上执行命令和断言。 语法简单但很强大，使您可以快速编写测试。


### Nightmare

[Nightmare](https://github.com/segmentio/nightmare) is a high-level browser automation library.


## 参考

[基于 React + Redux 的组件测试](http://front-ender.me/react/react-test-component.html)

[前端单元测试探索](https://segmentfault.com/a/1190000006933557)

[测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

[使用 Mocha + Chai + Sinon 测试 React + Redux 的 web 应用](http://zhaozhiming.github.io/blog/2015/12/19/use-mocha-and-chai-and-sinon-to-test-react-and-redux-webapp/)

[React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)

[测试 Redux 应用中使用 window.fetch 的 API 请求](http://frantic1048.logdown.com/posts/517113-test-redux-application-windowfetch-api-request)
