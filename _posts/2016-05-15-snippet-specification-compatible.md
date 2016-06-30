---
layout: post
title: 兼容多种模块规范
date: '2016-05-15 15:37:01 +0800'
categories: JavaScript
---

# 兼容多种模块规范

为了让同一个模块可以运行在前后端，以及兼容模块规范的环境，类库开发者需要将类库代码包装在一个闭包内。 以下代码能够就兼容 `Node`、`AMD`、`CMD`以及常见的`浏览器`环境。

```javascript
;(function (name, definition) {
  // 检查上下文环境是否为 AMD 或 CMD
  var hasDefine = typeof define === 'function',
    // 检查上下文环境是否为 Node
    hasExports = typeof module !== 'undefined' && module.exports;

  if (hasDefine) {
    // AMD 环境或 CMD 环境
    define(definition);
  } else if (hasExports) {
    // 定义为普通 Node 模块
    module.exports = definition();
  } else {
    // 将模块的执行结果挂在 window 变量中， 在浏览器中 this 指向 window 对象
    this[name] = definition();
  }
})('funcName', function () {
  // return funcDefinition
});
```

> 引用自 《深入浅出 Node.js》
