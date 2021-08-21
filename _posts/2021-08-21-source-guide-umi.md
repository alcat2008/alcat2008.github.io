---
layout: post
title: 源码指引之 - Umi
keywords: umi, 源码, 流程, overview, workflow
date: '2021-08-21 12:00:00 +0800'
keywords: umi
tags: [engineering]
categories: engineering
---

# 源码指引之 - Umi

本文仍然是工程化的主题，尝试去解构 Umi。[Umi](https://umijs.org/zh-CN){:target="_blank"} 是**插件化**的企业级前端应用框架，包含了工程化的方方面面，其核心就是：**微内核** + **插件**。

官方给出了两张图，对我们理解并使用 Umi 很有帮助。

- 插件体系

[![umi-plugin](/resources/umi_plugin.webp)](/resources/umi_plugin.webp)

- 插件的生命周期

[![umi-lifecycle](/resources/umi_lifecycle.webp)](/resources/umi_lifecycle.webp)

## 一图胜千言

Umi 代码版本号是 [v3.5.17](https://github.com/umijs/umi/tree/v3.5.17){:target="_blank"}

结合生命周期，分别对 Umi 源码进行调试并理解。这里多说两句，直接看 Umi 的源码可能比较懵，对于**运行时**，可以直接查看 `@/.umi` 代码目录下的内容，**编译时**多次调用 `generateXXXFile` 就是为了生成这些代码。

[![umi-workflow](/resources/workflow_umi.webp)](/resources/workflow_umi.webp)


## 推荐资料

- [蚂蚁前端研发最佳实践](https://github.com/sorrycc/blog/issues/90){:target="_blank"}
- [蚂蚁金服前端框架和工程化实践](https://www.infoq.cn/article/caxvurfin*dqvw4ieh1h){:target="_blank"}
