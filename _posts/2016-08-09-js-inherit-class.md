---
layout: post
title: JavaScript 继承和类
keywords: javascript, inherit, class, 继承, 类, prototype
description: JavaScript 继承机制详解，类的继承方式
date: '2016-08-09 08:00:00 +0800'
tags: [javascript]
categories: javascript
---

# JavaScript 继承和类

## 继承机制

### 构造函数 Constructor

为了解决从原型对象生成实例的问题，Javascript 提供了一个`构造函数（ Constructor ）`模式。

所谓"构造函数"，其实就是一个普通函数，但是内部使用了` this 变量`。对构造函数使用 new 运算符，就能生成实例，并且 this 变量会绑定在实例对象上。

使用构造函数生成的实例对象，会自动含有一个 constructor 属性，指向它们的构造函数。

Javascript 提供了一个 `instanceof` 运算符，验证原型对象与实例对象之间的关系。

### prototype

构造函数方法很好用，但是存在一个浪费内存的问题。

Javascript 规定，每一个构造函数都有一个 prototype 属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。

这意味着，所有实例对象需要共享的属性和方法，直接定义在 prototype 对象上；那些不需要共享的属性和方法，就放在构造函数里面。

由于所有的实例对象共享同一个 prototype 对象，那么从外界看起来，prototype 对象就好像是实例对象的原型，而实例对象则好像"继承"了 prototype 对象一样。

Javascript 继承机制的设计思想： `constructor + prototype`。

### \__proto__

一个对象的 \__proto__ 属性和内部属性 [[Prototype]] 指向一个相同的值 (`构造函数的 prototype 对象`)，原型的值可以是一个对象值也可以是 null (比如说 Object.prototype.\__proto__ 的值就是 null)。改变 \__proto__ 属性的值同时也会改变内部属性 [[Prototype]] 的值，除非该对象是不可扩展的。

- 所有构造器/函数的 \__proto__ 都指向 Function.prototype，它是一个空函数（Empty function）
- 所有对象的 \__proto__ 都指向其构造器的 prototype

### 继承方式

思路：**利用构造函数继承实例属性，利用原型链继承原型属性和方法**。

```javascript
function inherit(p) {
  if (p == null) throw TypeError();  // p 是一个对象，但不能是 null
  if (Object.create) return Object.create(p); // 若 Object.create 存在，则直接使用它

  var t = typeof p;
  if (t != "object" && t != "function") throw TypeError();
  function F() {}; // 定义空构造函数
  F.prototype = p; // 将其原型属性设置为 p
  return new F(); // 使用 f() 创建 p 的继承对象
}

function Parent() {}
function Child() {}

Child.prototype = inherit(Parent.prototype);
Child.prototype.constructor = Child;

var child = new Child()
child instanceof Child   // true
child instanceof Parent  // true

```

## 类 Class

从 ECMAScript 6 开始，JavaScript 中有了类（class）这个概念。但需要注意的是，这并不是说：JavaScript 从此变得像其它基于类的面向对象语言一样，有了一种全新的继承模型。JavaScript 中的类只是 JavaScript 现有的、基于原型的继承模型的一种语法包装，它能让我们用更简洁明了的语法实现继承。

ES6 并没有改变 JavaScript 基于原型的本质，只是在此之上提供了一些语法糖。class 就是其中之一，其它的还有 extends、super 和 static 。它们大多数都可以转换成等价的 ES5 语法。

```javascript
class Child extends Parent {
  constructor(firstName, lastName, age) {
    super(firstName, lastName)
    this.age = age
  }
}
```

以上 class 定义方式基本等价于：

```javascript
function Child(firstName, lastName, age) {
  Parent.call(this, firstName, lastName)
  this.age = age
}

Child.prototype = Object.create(Parent.prototype)
Child.constructor = Child
```

ES6 中的类实际上就是个函数，而且正如函数的定义方式有函数声明和函数表达式两种一样，类的定义方式也有两种，分别是：类声明、类表达式。

类声明和函数声明不同的一点是，函数声明存在变量提升现象，而类声明不会。也就是说，你必须先声明类，然后才能使用它，否则代码会抛出 ReferenceError 异常。

- 类的数据类型就是函数，类本身就指向构造函数。
- 类的所有方法都定义在类的 prototype 属性上。
- 类的内部所有定义的方法，都是不可枚举的。
- 类的属性名，可以采用表达式。

### constructor

constructor 方法是类的默认方法，通过 new 命令生成对象实例时，自动调用该方法。

类的构造函数，不使用 new 是没法调用的，会报错。这是它跟普通构造函数的一个主要区别，后者不用 new 也可以执行。

### Class 的继承

Class 之间通过 `extends` 关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

ES5 的继承，实质是先创造子类的实例对象 this，然后再将父类的方法添加到 this 上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先创造父类的实例对象 this（所以必须先调用 super 方法），然后再用子类的构造函数修改 this。

### 类的 prototype 属性和 \__proto__ 属性

Class 作为构造函数的语法糖，同时有 prototype 属性和 \__proto__ 属性，因此同时存在两条继承链。

- 子类的 \__proto__ 属性，表示构造函数的继承，总是指向父类。
- 子类 prototype 属性的 \__proto__ 属性，表示方法的继承，总是指向父类的 prototype 属性。

可以这样理解： 作为一个对象，子类的原型（\__proto__ 属性）是父类；作为一个构造函数，子类的原型（prototype 属性）是父类的实例。
