---
layout: post
title: 如何让 Finder 显示隐藏文件和文件夹
date: '2016-07-14 10:00:00 +0800'
categories: mac
---

# 如何让 Finder 显示/隐藏文件和文件夹

## 显示隐藏的文件和文件夹

1. 打开「Terminal」应用程序。

2. 输入如下命令：

> defaults write com.apple.finder AppleShowAllFiles -boolean true

> killall Finder

现在你将会在 Finder 窗口中看到那些隐藏的文件和文件夹了。

## 隐藏文件和文件夹

如果你想再次隐藏原本的隐藏文件和文件夹的话，执行如下命令：

> defaults write com.apple.finder AppleShowAllFiles -boolean false

> killall Finder
