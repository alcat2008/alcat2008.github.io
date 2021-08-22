---
layout: post
title: 前端工程化的理解
keywords: 前端, 工程化, engineering
description: 前端工程化的理解
date: '2017-06-28 11:00:00 +0800'
tags: [Architecture]
categories: architecture
---

# 前端工程化

本文尝试对前端工程化进行梳理，不当之处，欢迎讨论拍砖，谢谢！

--------------------------------------------------------------------------------

先从源头入手，前端工程化究竟要解决什么问题？答曰：`开发体验`！这里的开发泛指产品生命周期的开发、测试、部署等一系列过程。

那么，一个良好的开发体验需要包含哪些内容呢？

在我看来，就是兼顾 `开发效率` 的同时，`易于维护` 并且 `运维方便`。


## 开发效率

开发过程即产品的产出过程，所以其效率必然影响产品优劣。

### 开发流程

- 职责和协同
- 目标明确的任务、版本计划
- 敏捷开发
- 技术选型
- 自动创建项目、模块、页面、组件结构

### 开发工具

- 构建，webpack/browserify、grunt/gulp/node script
- 编译转换，babel、polyfill、shim
- Less / Sass -> CSS 编译
- CSS Autoprefixer 前缀自动补全
- 自动生成图片 CSS 属性，width & height 等
- CSS cssnano 压缩
- 自动生成雪碧图，自动多倍图
- 图片压缩
- 自定义图片转 base64
- JS、CSS、HTML 压缩
- 文件 Reversion (MD5) 指纹
- 兼容适配方案，px -> rem
- 监听文件变动，自动刷新浏览器 (LiveReload)
- Mock server

### 代码质量

- eslint, jslint
- lesshint, csslint, stylelint
- htmllint
- 测试，mocha/jest + enzyme + chai + sinon
- Code Review

### 性能优化

- 首屏渲染，服务端渲染
- 延迟、异步加载
- 静态文件压缩、合并
- DNS 预解析
- Cookie 优化
- 性能报表

### 安全可靠

- httponly, token 过滤
- https


## 易于维护

开发人员都是有始有终的，不是说功能完成了就不管了。你是团队的一员，会和别人协作，你会复用别人的代码，别人同样也会共用你的逻辑，所以如何做到与人方便也就是与己方便。

### 开发规范

- 代码规范
- 注释规范
- 命名规范
- 组件规范
- 版本控制规范

### 组件化、模块化

- 组件仓库


## 运维方便

运维的工作也可以用自动化手段实现。

### 项目部署

- CI/CD
- Jenkins
- 负载均衡，Nginx/HAProxy/Varnish Cache

### 监控

- 错误上报
- 日志
- 数据库


以上从 “自动化” 的角度罗列了工程化可能涉及到的方方面面，至于具体的技术选型本文不做讨论。
