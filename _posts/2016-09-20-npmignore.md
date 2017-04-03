---
layout: post
title: github 与 npm 合作神器之 .npmignore
keywords: .npmignore, github, npm
description: 什么是 .npmignore ？ 以及其在 github 与 npm 之间所扮演的角色。
date: '2016-09-20 11:00:00 +0800'
tags: [architecture]
categories: architecture
---

# 神奇的 .npmignore

常用 github 的人必然会用到 npm，那么疑问来了？基本上每个 npm 的 module 都关联了一个 github 项目，为何有的项目两者一致，而有的项目却相差甚远。特别是当自己想向 npm 发布 module 时，这个问题一直困扰着我。要是我不想将源码公开，或者 module 只包含编译后的代码，对使用者而言是不是更为合理。

往 github 上 commit 代码时有 .gitignore，是否也存在类似的文件控制 npm module 的上传选择？最终还是得到官网上寻觅，最终发现了它: .npmignore。

[keeping-files-out-of-your-package](https://docs.npmjs.com/misc/developers#keeping-files-out-of-your-package) 这部分内容详细阐述了 .npmignore 的用途及用法。

.npmignore 文件就是 .gitignore 文件的小弟，不仅语法相同，其功能也是针对 npm 而提供的扩充而已。

[what-is-an-npmignore-file-and-what-is-it-used-for](http://javascript.tutorialhorizon.com/2015/06/25/what-is-an-npmignore-file-and-what-is-it-used-for/) 这篇文章详细说明了 .npmignore 是用来干嘛的，非常简单，就不翻译了。
