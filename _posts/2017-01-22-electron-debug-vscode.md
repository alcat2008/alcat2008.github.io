---
layout: post
title: 使用 VSCode 调试基于 ES6 的 electron 主进程
keywords: electron, debug, es6, main, vscode
description: 使用 VSCode 调试 electron 主进程
date: '2017-01-22 11:00:00 +0800'
tags: [electron]
categories: architecture
---

# 使用 VSCode 调试 electron 主进程

## 1. 在 VSCode 中打开一个 Electron 工程。

```bash
$ git clone https://github.com/alcat2008/electron-react-boilerplate.git
$ code electron-react-boilerplate
```

## 2. 添加一个文件 `.vscode/launch.json` 并使用一下配置：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Main Process",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron",
      "program": "${workspaceRoot}/src/main.development.js",
      "runtimeArgs": [
          "-r",
          "babel-register",
          "-r",
          "babel-polyfill"
      ],
      "env": {
          "HOT": "1",
          "NODE_ENV": "development"
      },
    }
  ]
}
```

**注意：** 在 Windows 中，`runtimeExecutable` 的参数是 `"${workspaceRoot}/node_modules/.bin/electron.cmd"`。

## 3. 调试

在 `main.development.js` 设置断点， 并在 [Debug View](https://code.visualstudio.com/docs/editor/debugging) 中启动调试。你应该能够捕获断点信息。

## 参考文章

- [使用 VSCode 进行主进程调试](https://github.com/electron/electron/blob/master/docs-translations/zh-CN/tutorial/debugging-main-process-vscode.md)
- [vscode debug ES6 application](http://stackoverflow.com/questions/31711286/vscode-debug-es6-application)
