---
layout: post
title: 重构 React Native 项目
date: '2016-06-13 15:37:01 +0800'
tags: [React Native]
categories: react-native
---

# Refactor for ios

## 原因

- 目前 React Native 平均两周更新一个版本，项目当前的一些bug可以通过升级解决，同时，新版本也提供了更为丰富的API以及更好的开发、设计体验。
- 考虑到项目中有很多自定义的组件，若通过传统的方式（react-native init 模板）去升级，每次都要将自定义的组件重新添加，很麻烦。

## 目标

- 升级 React Native 版本
- 变更项目结构，将 React Native 作为项目的依赖，方便后期的版本升级

## 步骤

新建 native 项目，然后将现有项目作为 view 显示， React Native 作为 CocoaPods 的依赖库引入。

## 重大变更

- 从0.18开始， React Native 默认项目全面转向ES6
- 0.25版本之后，在 React Native 中引用 React 的做法发生了变更

```javascript
import React, { Component, View } from 'react-native';
```

应变为

```javascript
import React, { Component } from 'react';
import { View } from 'react-native';
```

属于 'react' 的组件:

```javascript
Children
Component
PropTypes
createElement
cloneElement
isValidElement
createClass
createFactory
createMixin
```

属于 'react-native' 的组件:

```javascript
hasReactNativeInitialized
findNodeHandle
render
unmountComponentAtNode
unmountComponentAtNodeAndRemoveContainer
unstable_batchedUpdates
View
Text
ListView
...
and all the other native components.
```

可以使用 [codemod-RN24-to-RN25](https://github.com/sibelius/codemod-RN24-to-RN25) 自动化重写项目的 imports。
