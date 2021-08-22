---
layout: post
title: style 模块化思考
keywords:
description:
date: '2016-09-28 11:00:00 +0800'
tags: [React, CSS]
categories: architecture
---


# style 模块化思考

style 即样式，包含三层含义：

- Layout 布局 — 组件/元素间的关联关系
- Appearance 外观 — 组件/元素的外在特征
- Behavior and state 行为和状态 — 组件/元素不同状态下的展现

## css 模块化存在的问题

- 全局污染
- 命名混乱
- 依赖管理不彻底
- 无法共享变量
- 代码压缩不彻底


## React style

### Layout

组件中不要包括布局样式，用布局组件包装该组件。

```javascript
// This couples your component to the layout system
// It reduces the reusability of your component
<UserBadge
 className="col-xs-12 col-sm-6 col-md-8"
 firstName="Michael"
 lastName="Chan" />

// This is much easier to maintain and change
<div class="col-xs-12 col-sm-6 col-md-8">
  <UserBadge
   firstName="Michael"
   lastName="Chan" />
</div>
```

为了更好的支持布局，将组件设计为 `100%` 的 `width` 和 `height`。

## Appearance

此处争议极大，关键在于是否采用 “内联样式”。

应当取决于组件的设计，以及团队对 JavaScript 的舒适度。


比较好的方案：

- 能使用 css 的预处理器工具，如 postcss
- 全局变量
- 局部样式（全局 + css-modules）   --- 唯一前缀

最终方案为： `css-module + less`


## css-modules

### 外部如何覆盖局部样式

采用 css-modules 生成混淆的 class 名后，可以解决命名冲突，但因为无法预知最终 class 名，不能通过一般选择器覆盖。我们现在项目中的实践是可以给组件关键节点加上 `data-role` 属性，然后通过属性选择器来覆盖样式。


```javascript
// dialog.js
return <div className={styles.root} data-role="dialog-root">
  <a className={styles.disabledConfirm} data-role="dialog-confirm-btn">Confirm</a>
  ...
</div>

/* dialog.css */
[data-role="dialog-root"] {
  // override style
}
```

### 如何与全局样式共存

前端项目不可避免会引入 normalize.css 或其它一类全局 css 文件。使用 Webpack 可以让全局样式和 CSS Modules 的局部样式和谐共存。

```javascript
// webpack.config.js 局部
module: {
  loaders: [{
    test: /\.jsx?$/,
    loader: 'babel'
  }, {
    test: /\.scss$/,
    exclude: path.resolve(__dirname, 'src/styles'),
    loader: 'style!css?modules&localIdentName=[name]__[local]!sass?sourceMap=true'
  }, {
    test: /\.scss$/,
    include: path.resolve(__dirname, 'src/styles'),
    loader: 'style!css!sass?sourceMap=true'
  }]
}

/* src/app.js */
import './styles/app.scss';
import Component from './view/Component'

/* src/views/Component.js */
// 以下为组件相关样式
import './Component.scss';
```

目录结构如下：

```
src
├── app.js
├── styles
│   ├── app.scss
│   └── normalize.scss
└── views
    ├── Component.js
    └── Component.scss
```

这样所有全局的样式都放到 `src/styles/app.scss` 中引入就可以了。其它所有目录包括 `src/views` 中的样式都是局部的。


## 参考

- [CSS Modules 详解及 React 中实践](https://zhuanlan.zhihu.com/p/20495964?refer=purerender)
- [CSS 模块](http://www.75team.com/post/1049.html)
