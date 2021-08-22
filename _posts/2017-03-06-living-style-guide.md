---
layout: post
title: Living style guide
keywords: Living style guide, UI libraries
description: Living style guide
date: '2017-03-06 09:00:00 +0800'
tags: [Architecture]
categories: architecture
---

# Living style guide

## What ?

`Living style guide` 是什么呢？没有找到合适的中文翻译，也没有恰当的中文解释。

> A **living style guide** is a **documentation of UI elements and patterns** collected from a site or application with the purpose to allow developers to use **consistent styles across their whole project**.

## 作用

在我的理解，按照字面意思，`living style guide` 就是样式指南集，不过是动态生成的。它摆脱了传统的手工模式，一旦你按照某种既定格式书写文档，将自动生成网站，用于展示相应组件或样式的使用说明。而这种文档格式，目前比较流行的就是 Markdown。

所以 `living style guide` 不仅仅是开发便利，更是一整套构建工具，服务于软件项目的方方面面。需求、设计都可以参考此种方式。

## 构建

`living style guide` 是需求、开发、设计等角色间共同的绝佳工具，构建路径很清晰。

```
一份包含示例代码、API 说明文档的 Markdown => 一个网站（代码及演示）
```

如果你还不明白，看看 [ant.desigh](https://ant.design/docs/react/introduce-cn)，随便打开一个组件，这就是我心目中理想的 `living style guide` - `好写`、`好用`、`好看`。

那么实现工具呢？[bisheng](https://github.com/benjycui/bisheng)

如果你的团队有类似的工具，那么你是幸运的。如果没有，还等什么，`Just do it !`。

## 参考

[Ten Living Style Guide Tools for Web Designers – Best of](http://www.hongkiat.com/blog/best-living-style-guide-tools/)
