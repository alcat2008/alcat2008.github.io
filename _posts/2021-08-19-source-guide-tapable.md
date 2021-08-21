---
layout: post
title: 源码指引之 - Tapable
keywords: tapable, 源码, 流程, overview, workflow
date: '2021-08-19 22:00:00 +0800'
keywords: tapable
tags: [engineering]
categories: engineering
---

# 源码指引之 - Tapable

我现在所使用的工程化方案都是基于 Umi，Umi 最终会调用 Webpack。Umi 和 Webpack 内部的事件流都是基于 Tapable，所以先理解了 Tapable，再去看 Umi 和 Webpack 就能事半功倍了。

## 一图胜千言：

Tapable 代码版本号是 [v2.2.0](https://github.com/webpack/tapable/tree/v2.2.0){:target="_blank"}

[![tapable-workflow](/resources/workflow_tapable.webp)](/resources/workflow_tapable.webp)

## 推荐资料

- [webpack核心模块tapable源码解析](https://www.cnblogs.com/dennisj/p/14606902.html){:target="_blank"}
