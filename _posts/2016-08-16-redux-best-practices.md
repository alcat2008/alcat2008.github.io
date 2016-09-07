---
layout: post
title: Redux 最佳实践
date: '2016-08-16 08:00:00 +0800'
categories: architecture
---

本文的灵感来自于 Redux 官网的 issue： [Recommendations for best practices regarding action-creators, reducers, and selectors](https://github.com/reactjs/redux/issues/1171)。总结了讨论内容的基础上并不断完善，以期为 Redux 使用者提供思路，构建更为健壮的应用。

- Use selectors everywhere

- Do more in action-creators and less in reducers

- Use ImmutableJS

- Do not nest objects in mapStateToProps and mapDispatchToProps
