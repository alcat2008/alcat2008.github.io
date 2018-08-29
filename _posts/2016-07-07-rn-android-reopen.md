---
layout: post
title: RN 应用 Android 版本首次安装后重新加载的问题
date: '2016-07-07 10:00:00 +0800'
tags: [react-native]
categories: react-native
---

# 问题

React Native 开发的 APP，首次安装完成后在安装界面直接"打开"应用，再按 Home 键返回桌面，重新进入 APP，会重新实例化 Launcher Activity。

# 参考

- <http://blog.csdn.net/love100628/article/details/43238135>
- <http://stackoverflow.com/questions/3042420/home-key-press-behaviour/4782423>

# 解决方案

参考文档里都是在 Android 原生项目中多个 Actiivity 跳转时出现的问题，虽然 React Native 里只用了一个 Activity，但还是重新渲染了，所以归根到底，问题是一样的，肯定和加载机制脱不了关系。

研究 Android 的 `launchMode`，参考文章 <http://blog.csdn.net/liuhe688/article/details/6754323>

应该知道是什么原因引起了吧，最终的解决方案如下：

```
<activity
  android:name=".MainActivity"
  android:launchMode="singleTop" // 加入此行
  ...
</activity>
```
