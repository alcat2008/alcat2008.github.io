---
layout: post
title: JavaScript 判等你真的清楚吗？
date: '2016-07-19 08:00:00 +0800'
tags: [javascript]
categories: javascript
---

# JavaScript 判等

提到 JavaScript 判等，很多人就自诩神秘的提到，是说 `==` 和 `===` 对吧，区别是。。。

5分钟以前我也是这样想的，关于 `==` 和 `===` 的区别，网上一搜一大把，总结下答案基本一致：

> "==="叫做严格运算符，"=="叫做相等运算符。

> 双等号会造成类型转换，推荐一律使用三等号。

先不提是不是简单（片面），看看很多新手写的代码，通篇的 `==`，能想到 `===` 也是不小的进步了。

一度以为自己对 JavaScript 判等不说深入了解，至少不会用错，直到遇到了这篇文章 [20个 Js 变态题解析](https://segmentfault.com/a/1190000005988554)。

## 面试题

第一题

> console.log([]==[]);

第二题

> console.info(2 == [2]);

就只说这两题，答案是多少？

... 漫长的等待 ...

答案揭晓，第一题： `false` ； 第二题： `true`

如果答错了，那下面的内容就很有必要仔细阅读了。

## == VS ===

1. 对于string,number等基础类型，`==` 和 `===` 是有区别的

  - 不同类型间比较，`==` 只比较 **转化成同一类型后的值** 看 **值** 是否相等，`===` 如果类型不同，其结果就是不等

  - 同类型比较，直接进行 **值** 比较，两者结果一样

2. 对于Array, Object等高级类型，`==` 和 `===` 是没有区别的, 进行 **指针地址** 比较

3. 基础类型与高级类型，`==` 和 `===` 是有区别的

  - 对于 `==`，将高级转化为基础类型，进行 **值** 比较

  - 因为类型不同，`===` 结果为false

## 解答

看了上一节的内容，应该明白了吧。

第一题

```javascript
/*
  console.info([] instanceof Array); // true
  console.info([] instanceof Object); // true
  [] 是一个数组对象. [] == [] 等价于：
  var a = [];
  var b = [];
  a == b; 所以肯定是false
*/
console.log([]==[]); // false
```

第二题

```javascript
/*
  再看一遍上一节的内容。
  最终都转化成 “值” 比较
*/
console.info(2 == [2]); // true
```

## '规则'

[通过一张简单的图，让你彻底地、永久地搞懂JS的==运算](https://segmentfault.com/a/1190000006012804) 一文的结尾处总结了 `==` 的规则，还是非常准确的。

- `undefined == null`，结果是true。且它俩与所有其他值比较的结果都是false。
- `String == Boolean`，需要两个操作数同时转为Number。
- `String/Boolean == Number`，需要String/Boolean转为Number。
- `Object == Primitive`，需要Object转为Primitive(具体通过valueOf和toString方法)。

## 参考

若多此文内容还有疑问，或者想深入学习，请参考知乎讨论 [Javascript 中 == 和 === 区别是什么？](https://www.zhihu.com/question/31442029)
