---
layout: post
title: JavaScript 函数编程
keywords: javascript, function, 函数, functional, 函数式
description: JavaScript 函数编程相关内容深入分析，包括作用域、箭头函数、函数式编程等
date: '2016-08-04 08:00:00 +0800'
tags: [JavaScript]
categories: javascript
---

# JavaScript 函数编程

JavaScript 函数编程所涉及的内容既有深度也有广度，所以本文从函数入手，并逐一讨论，最终过度到函数式编程这一终极大法器。

## Programming

Generally, the craft of programming is the factoring of a set of requirements into a set of `functions` and `data structures`.

## Functions

众所周知，在 JavaScript 中，函数是一等公民。

```
A function encloses a set of statements.
Functions are the fundamental modular unit of JavaScript.
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

## Scope

### Lexical Scope 词法作用域

词法分析过程 Lex-time，是指系统将源码字符序列转换为标记（token）序列的过程。词法作用域就是说在词法分析过程中指派的作用域，词法作用域在词法解析过程中就已经指定了。

JavaScript 是基于词法作用域的语言，所以通过阅读包含变量定义在内的数行源码就可以确定变量的作用域。

与词法作用域相对应的是 `动态作用域 Dynamic Scope`，其作用域是在代码运行时定义的，而非代码解析时。动态作用域只关心它在哪里调用，并不关心它本身是怎样或者在哪里声明。动态作用域的域链基于调用栈，而不是代码中的嵌套关系。

举个栗子：

```javascript
function foo() {
    console.log( a );
}

function bar() {
    var a = 3;
    foo();
}

var a = 2;

bar();
```

最终的结果是：2。因为 JavaScript 采用词法作用域，foo 被解析的时候是在全局作用域，所以 a 是全局作用域中的 2，而非 bar 里面的 a。假设 JavaScript 采用的是动态作用域，foo 是在 bar 中被调用，那么 a 最终查找到的是 bar 作用域里的 3。

Javascript 中的函数 **“在定义它们的作用域里运行，而不是在执行它们的作用域里运行”**，这是权威指南里抽象而精辟的总结。

尽管 JavaScript 采用词法作用域，但是 this 却是采用类似的动态作用域，与调用位置有关。

### Scope Chain 作用域链

每一段 JavaScript 代码都有一个与之关联的作用域链。这个作用域链是一个对象列表或者链表，定义了这段代码 “作用域中” 的变量。

当定义一个函数时，它实际上保存一个作用域链。当调用这个函数时，它创建一个新的对象来存储它的局部变量，并将这个对象添加至保存的那个作用域链上，同时创建一个新的更长的表示函数调用作用域的 “链”。对于嵌套函数来讲，事情变得更加有趣，每次调用外部函数时，内部函数又会重新定义一遍。因为每次调用外部函数的时候，作用域链都是不同的。


## Arrow Functions

function 函数总会自动接收一个 this 值。你是否写过这样的hack代码：

```javascript
{
  ...
  addAll: function addAll(pieces) {
    var self = this;
    _.each(pieces, function (piece) {
      self.add(piece);
    });
  },
  ...
}
```

在这里，你希望在内层函数里写的是 this.add(piece)，不幸的是，内层函数并未从外层函数继承 this 的值。在内层函数里，this会是 window 或 undefined（根据 ES3 和非严格的 ES5 对函数调用的规定，调用上下文是全局对象。然而在严格模式下，调用上下文是 undefined。），临时变量 self 用来将外部的 this 值导入内部函数。（另一种方式是在内部函数上执行 .bind(this)，两种方法都不甚美观。）

ES6 中引入了箭头函数，和普通 function 函数相比，**箭头函数没有它自己的this值**，箭头函数内的this值继承自外围作用域。

在ES6中，不需要 hack this了，但你需要遵循以下规则：

- 通过 object.method() 语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的 this 值。
- 箭头函数不能作为 generator 函数使用。
- 其它情况全都使用箭头函数。

```javascript
// ES6
{
  ...
  addAll: function addAll(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}
```

在 ES6 的版本中，注意 addAll 方法从它的调用者处获取了 this 值，内部函数是一个箭头函数，所以它继承了外围作用域的 this 值。

超赞的是，在ES6中你可以用更简洁的方式编写对象字面量中的方法，所以上面这段代码可以简化成：

```javascript
// ES6的方法语法
{
  ...
  addAll(pieces) {
    _.each(pieces, piece => this.add(piece));
  },
  ...
}
```

箭头函数与非箭头函数间还有一个细微的区别，箭头函数不会获取它们自己的 arguments 对象。所以在 ES6 中，你可能更多地会使用不定参数和默认参数值这些新特性。

## Curry & Thunk

关于柯里化 Curry，请参考另一篇文章：[柯里化 Curry](/javascript/curry.html)

至于 Thunk，个人认为是 Curry 的子集。在 JavaScript 语言中，Thunk 函数将多参数函数替换成单参数的版本，且 **只接受回调函数作为参数**。关于 Thunk 的详细介绍可参考文章 [Thunk 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/thunk.html)。


## Functional Programming 函数式编程

"函数式编程"是一种"编程范式"（programming paradigm），也就是如何编写程序的方法论。它属于"结构化编程"的一种，主要思想是把运算过程尽量写成一系列嵌套的函数调用。

函数式编程具有五个鲜明的特点：

- 函数是"第一等公民"
- 只用"表达式"，不用"语句"
- 没有"副作用"
- 不修改状态
- 引用透明

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


## 参考文献

- 《 Javascript 权威指南 》
- [You-Dont-Know-JS - 词法作用域](https://segmentfault.com/a/1190000002532217)
- [再探 Javascript 词法作用域](http://www.cnblogs.com/mophee/archive/2009/03/15/1412590.html)
