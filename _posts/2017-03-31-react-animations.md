---
layout: post
title: CSS 动画及其在 React 中的应用
keywords: react, css, animation, transition, transform, ReactCSSTransitionGroup, react-addons-css-transition-group, Animate.css
description: 介绍了 CSS 动画、 Animate.css 动画库、以及在 React 中使用 react-addons-css-transition-group 插件实现动画方法
date: '2017-03-31 11:00:00 +0800'
tags: [React, CSS]
categories: react
---

# CSS 动画及其在 React 中的应用


## CSS 动画

CSS 中涉及 “动” 的属性主要有 transform、transition 和 animation 三个，理清这三者之间的关系，基本上也就能明白如何去实现动画效果了。

### Transform - 2D/3D 转换属性

transform 属性很好理解，就是向元素应用 2D 或 3D 转换，该属性允许我们对元素进行旋转、缩放、移动或倾斜。示例： [扔到桌子上面的图片](http://www.w3school.com.cn/tiy/t.asp?f=css3_image_gallery)

需要注意的是，transform 属性是静态属性，一旦写到 style 里面，将会直接显示作用的结果，无任何变化过程。

### Transition - 过渡属性

准确的说，transition 是 `过渡` 属性，是元素从一种样式逐渐改变为另一种的效果。那么什么是动画呢，不就是在过渡过程中的一系列效果集合吗，所以不管从定义、实现，亦或是呈现效果，transition 都称得上是动画。示例： [向宽度、高度和转换添加过渡效果](http://www.w3school.com.cn/tiy/t.asp?f=css3_transition2)

transition 非常简单易用，但是需要事件触发，并且是一次性的，过渡状态也相对单一。

### Animation - 动画属性

animation 是官方专属动画属性，对 transition 功能进行增强扩展，同时还提供了 keyframes 关键字用来定义动画的各个状态。示例： [调皮的小方块](http://www.w3school.com.cn/tiy/t.asp?f=css3_animation5)

@keyframes 精确的控制动画变化中时间轴上的关键帧属性效果，用百分比来规定变化发生的时间，或用关键词 "from" 和 "to"，等同于 0% 和 100%。0% 是动画的开始，100% 是动画的完成。

另外在 animation 属性里面还有一个非常重要的动画属性就是：animation-fill-mode，这个属性规定对象动画时间之外的状态，指定动画执行前后如何为目标元素应用样式。这个很方便我们控制动画的结尾样式，保证动画的整体连贯。


## Animate.css

[Animate.css](https://daneden.github.io/animate.css/) 是一个动画库，里面预设了很多种常用的动画，包括抖动（shake）、闪烁（flash）、弹跳（bounce）、翻转（flip）、旋转（rotateIn/rotateOut）、淡入淡出（fadeIn/fadeOut）等多种动画效果，几乎包含了所有常见的动画效果。

使用非常方便，就是普通的 class，比如实现一个无线循环的弹跳效果：

```javascript
<h1 class="animated infinite bounce">Example</h1>
```

## ReactCSSTransitionGroup

React 官方提供了两个插件用于处理动画效果：一个是偏底层的 `react-addons-transition-group`，它有一个完整的生命周期来驱动动画的完成；另一个是在前者基础上进一步封装的 `react-addons-css-transition-group`。本文仅介绍 `react-addons-css-transition-group` 的使用方式，结合 Animate.css，可以快速实现组件的各种动画。

比如为组件添加淡入淡出的动画：

```javascript
import ReactCSSTransitionGroup from 'react-addons-css-transition-group';

class AnimationDemo extends React.Component {
  render() {
    const childrenComponent = (
      <div key={...} className="animated">我是动画</div>
    );

    return (
      <ReactCSSTransitionGroup
        transitionEnter={true}
        transitionLeave={true}
        transitionEnterTimeout={500}
        transitionLeaveTimeout={500}
        transitionName={{
          enter: 'fadeInUp',
          leave: 'fadeOut'
        }}
      >
        {childrenComponent}
      </ReactCSSTransitionGroup>
    );
  }
}
```

简单介绍下各个属性：

- `transitionAppear`/`transitionEnter`/`transitionLeave` Bool 类型，标识是否开启动画
- `transitionAppearTimeout`/`transitionEnterTimeout`/`transitionLeaveTimeout` 动画的时长
- `transitionName` 自定义各个动画的 CSS 样式

如你所见，在 React 组件上应用 Animate.css 的动画就是这么简单，当然你也可以用自定义的样式。详细的属性介绍请参考官网的说明 [React Animation Add-Ons](https://facebook.github.io/react/docs/animation.html)。

使用 `react-addons-css-transition-group` 创建动画有几点需要注意：

- **组件必须设定独一无二的 key 值**

  官网上特意强调了这句话：

  > You must provide the key attribute for all children of ReactCSSTransitionGroup, even when only rendering a single item. This is how React will determine which children have entered, left, or stayed.

- **必须是挂载的组件才能实现动画**

  也就是说像以下的实现方式是没有动画效果的。

  ```javascript
  <ReactCSSTransitionGroup
    transitionEnter={true}
    transitionLeave={true}
    transitionEnterTimeout={500}
    transitionLeaveTimeout={500}
    transitionName={{
      enter: 'fadeInUp',
      leave: 'fadeOut'
    }}
  >
    {datalist.map(item => (
      <div>item<div>
    ))}
  </ReactCSSTransitionGroup>
  ```

此外，在结合 Animate.css 实现动画效果时还需要注意的是，`animated` 的默认实现中：

```javascript
.animated {
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}
```

默认的 animation-duration 动画时间都是 1s，所以如果 ReactCSSTransitionGroup 的 transitionXXXTimeout 时间设置小于 1s，有可能看到的动画不连贯，因为动画过程被硬生生的截断了。同样的问题还可能在以下几个动画效果中出现：

```javascript
.animated.hinge {
  -webkit-animation-duration: 2s;
  animation-duration: 2s;
}

.animated.flipOutX,
.animated.flipOutY,
.animated.bounceIn,
.animated.bounceOut {
  -webkit-animation-duration: .75s;
  animation-duration: .75s;
}
```

## 结语

本文简单介绍了 CSS 动画、 Animate.css 动画库、以及在 React 中使用 `react-addons-css-transition-group` 插件实现动画方法。

如果想要更炫酷的效果，可以参考 [react-motion](https://github.com/chenglou/react-motion)、[velocity-react](https://github.com/twitter-fabric/velocity-react)、[rc-animate](https://github.com/react-component/animate)、[Ant Motion](https://motion.ant.design/) 。


## 参考资料

[CSS3 动画](http://www.w3school.com.cn/css3/css3_animation.asp)

[CSS Transform / Transition / Animation 属性的区别](http://blog.iwege.com/posts/the-different-between-transform-transition-animation.html)

[React Animation Add-Ons](https://facebook.github.io/react/docs/animation.html)

[React 动画](http://pinggod.com/2016/React-%E5%8A%A8%E7%94%BB/)
