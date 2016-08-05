---
layout: post
title: JavaScript 函数式编程
date: '2016-08-04 08:00:00 +0800'
categories: javascript
---

## Programming

Generally, the craft of programming is the factoring of a set of requirements into a set of `functions` and `data structures`.

## Functions

众所周知，在 JavaScript 中，函数是一等公民。

```
A function encloses a set of statements. Functions are the fundamental modular unit of JavaScript.
They are used for code reuse, information hiding, and composition.
Functions are used to specify the behavior of objects.
```

函数就是对象，特殊之处在于它们可以被`调用`。

JavaScript 中函数有4种调用模式，区别在于对关键参数 `this` 的初始化方式。

- **method** invocation pattern
- **function** invocation pattern
- **constructor** invocation pattern
- **apply** invocation pattern

函数总是有返回值，如果没有指定，则返回 `undefined`。



## 延伸阅读

- Monads

  [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

  中文版 [图解 Monad](http://www.ruanyifeng.com/blog/2015/07/monad.html)

- Functional Programming

  [mostly-adequate-guide/details](https://drboolean.gitbooks.io/mostly-adequate-guide/content/)

  中文版 [JS函数式编程指南](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)

- React Basic Theoretical Concepts

  [React - Basic Theoretical Concepts](https://github.com/reactjs/react-basic)

  中文版 [React 设计思想](https://github.com/react-guide/react-basic)
