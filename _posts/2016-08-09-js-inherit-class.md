---
layout: post
title: JavaScript 继承和类
date: '2016-08-09 08:00:00 +0800'
categories: javascript
---

# JavaScript 继承和类

## 继承机制

### 构造函数 Constructor

为了解决从原型对象生成实例的问题，Javascript 提供了一个`构造函数（ Constructor ）`模式。

所谓"构造函数"，其实就是一个普通函数，但是内部使用了` this 变量`。对构造函数使用 new 运算符，就能生成实例，并且 this 变量会绑定在实例对象上。

使用构造函数生成的实例对象，会自动含有一个constructor属性，指向它们的构造函数。

Javascript 提供了一个 `instanceof` 运算符，验证原型对象与实例对象之间的关系。

### prototype

构造函数方法很好用，但是存在一个浪费内存的问题。

Javascript 规定，每一个构造函数都有一个 prototype 属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。

这意味着，所有实例对象需要共享的属性和方法，直接定义在 prototype 对象上；那些不需要共享的属性和方法，就放在构造函数里面。

由于所有的实例对象共享同一个prototype对象，那么从外界看起来，prototype 对象就好像是实例对象的原型，而实例对象则好像"继承"了 prototype 对象一样。

Javascript 继承机制的设计思想： `constructor + prototype`。
