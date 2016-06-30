---
layout: post
title: JavaScript 编码规范
date: '2016-05-19 15:37:01 +0800'
categories: JavaScript
---

# 编码规范

在团队协作过程中，编码规范的作用想必不用多说，本文主要介绍如何选择及相应的辅助工具，最大程度上保证代码质量。

## 选择标准

- 认可度高，该规范现在已经或者即将成为国际 JavaScript 标准了
- 支持项目的技术选型
- 是否提供插件支持

参考以上几条标准， airbnb 团队在 GitHub 维护的 [airbnb/javascript](https://github.com/airbnb/javascript) 瞬间脱颖而出。

- 目前 GitHub 上最有热度的规范，有恐怖的 35000+ 个 star
- 支持 ES6、React 等
- 官方提供 ESlint、JSCS 插件支持

## 规则配置

airbnb 团队在分享编码风格的同时，还提供了与之对应的 eslint 配置文件。他们提供的配置文件，根据使用环境不同，分三种：

1、 `eslint-config-airbnb`

支持`EcmaScript 6+`和`React`

- `npm install --save-dev eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-jsx-a11y eslint`
- add `"extends": "airbnb"` to your `.eslintrc`

2、 `eslint-config-airbnb-base`

支持`ECMAScript 6+`

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`
- add `"extends": "airbnb-base"` to your `.eslintrc`

3、 `eslint-config-airbnb-base/legacy`

支持`ES5`

- `npm install --save-dev eslint-config-airbnb-base eslint-plugin-import eslint`
- add `"extends": "airbnb-base/legacy"`to your `.eslintrc`

## Eslint Parser

如果程序中还有ES5的写法，建议使用 'babel-eslint' 作为 eslint 的 parser，只要在 `.eslintrc` 中加入如下代码即可。

```javascript
"parser": "babel-eslint"
```

## 可视化提示

借助 airbnb 提供的 ESlint 插件， 如果使用 `Atom` 进行开发，可以利用 `linter` 和 `linter-eslint` 插件进行可视化错误提示。

## 代码自动格式化

我们希望有一个自动工具能帮我们把代码进行自动格式成符合我们代码规范的代码。我们使用的是 `esformatter`，它同样有 `Atom` 的插件工具。这个插件可以让我们在保存页面的时候自动格式化代码。 esformatter 和 ESlint 同样基于 espree开发，所以他们的配置都保持一致。
