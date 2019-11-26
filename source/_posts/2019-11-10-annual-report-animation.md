---
title: 年报系列分享之关键帧动画篇
---

在CSS3中，做动画常用的方法有两种，一个是`transition`，第二个是`animation`。其中`transition`直接翻译成中文就是“过渡”，意思是元素的某个属性从一个值变成另一个值时采用渐变的方式，而不是直接修改。因此并不是所有属性都能使用渐变动画，只有那些值为数字或可以转换成数字的属性才可以使用渐变动画，比如`width`，`height`等数字类型的属性，比如`transform`中的`translate`，`scale`和`rotate`等，再比如`color`和`background-color`等可以转换成数值的颜色值也可以。但是`transition`有以下限制：

1. 需要触发事件，`transition`声明的是从一个值变成另一个值时的渐变效果，只有在从一个值变成另一个值时才有动画效果，比如用户行为点击（`:focus`, `:checked`），鼠标虚浮（`:hover`）等，或`DOM`操作。
2. 只能指定结束状态，不能精确指定动画过程中每一帧的细节，只能通过过渡函数`transition-timing-function`进行粗略设置。
3. 既然是从一个值变成另一个值，变成目标值以后就结束了，它不能重复触发。

为了突破以上限制，我们就要祭出CSS动画的大杀器`animation`了。

## Animation 属性详解

`animation`属性是`animation-name`，`animation-duration`, `animation-timing-function`，`animation-delay`，`animation-iteration-count`，`animation-direction`，`animation-fill-mode` 和 `animation-play-state` 属性的一个简写形式。
下面我们分别来看每一个属性的含义：
1. `animation-name` 属性代表动画名称，由`@keyframes`定义的一个帧动画。关键字`none`表示没有帧动画，可以在不改变其他属性的情况下使动画失效。
2. `animation-duration` 属性指定一个动画周期的持续时长，默认为0，表示无动画效果。
3. `animation-timing-function` 属性指定一个动画周期中执行的节奏，比如`ease`（默认值），`ease-in`，`ease-out`，`ease-in-out`，`linear`（匀速），`cubic-bezier(n,n,n,n)`等，还可以通过`steps`函数制作阶梯动画，详细请参考张鑫旭的文章[《CSS3 animation属性中的steps功能符深入介绍》](https://www.zhangxinxu.com/wordpress/2018/06/css3-animation-steps-step-start-end/)。
4. `animation-delay` 属性指定动画应用到元素之后延迟多久开始执行，默认值为`0s`即立即执行。
5. `animation-iteration-count` 属性指定动画执行的次数，可以是1次，2次，也可以是无限次（`infinite`），还可以是小数，表示执行到动画周期的中间某个位置停止。
6. `animation-direction` 属性指示动画是否反向播放。可选值包括`normal`（默认值，每个动画周期都从头开始），`alternate`（交替，正反正反...方向执行动画），`reverse`（每个动画周期都从尾开始），`alternate-reverse`（反向交替，反正反正...方向执行动画）。
7. `animation-fill-mode` 设置CSS动画在执行之前和之后如何将样式应用于其目标元素。可选值包括`none`（默认值，动画执行前样式不会应用到元素上），`forwards`（动画执行结束时，最后一帧的样式应用到元素上）`backwards`（动画应用到目标元素时，立即将第一帧的样式应用到元素上，并且在`animation-delay`期间保持）`both`（同时遵循`forwards`和`backwards`的规则）。
8. `animation-play-state` 指定动画是否正在执行，可选值包括`running`（初始值，动画正在执行）和`paused`（动画已经中止），通过该属性可以实现当鼠标悬浮时元素静止，鼠标离开之后继续动画的效果。

`animation`属性过于复杂，以至于仅仅是简单罗列都要花费大量篇幅。当然我不是为了解释以上`animation`的每个属性来写这篇博客的，而是在做年报项目时遇到了一个很常见的动画效果，但是实现起来却很复杂，而且不利于复用。动画效果就是一朵云从屏幕中间某个位置开始匀速飘向屏幕左侧，到屏幕左侧后再从右侧开始匀速飘向屏幕中间，周而复始，持续匀速漂移。如下图，

![](http://qiniu.wecode.club/blog/cloud-moving.gif)

根据前面对动画属性的介绍，不难看出，匀速对应的是`animation-timing-function: linear;`，周而复始对应的是`animation-iteration-count: infinite;`，因此对于一个从屏幕中间位置开始往左移动的云朵，假设云朵的起始位置是30vw，宽度是10vw，到达左侧完全看不见时移动的距离是30vw + 10vw = 40vw，而整个动画周期移动的距离是100vw + 10vw = 110vw; 因此到达左侧完全看不见时的时间处在整个动画周期的 40vw / 110vw = 36.36% 处，之后我们在 36.37% 让其瞬间转移到右侧并继续匀速左移，知道再次移动到起始位置时，一个动画周期结束。代码如下：

```css
.cloud {
    animation: cloud-move 20s linear infinite;
}

@keyframes cloud-move {
    0% { transform: translateX(30vw); }
    36.36% { transform: translateX(-10vw); }
    36.37% { transform: translateX(100vw); }
    100% { transform: translateX(30vw); }
}
```

单纯地说上述动画并不算难，但是如果页面中有多个云朵，为了更加逼真和布局效果，云朵的起始位置（X轴）均不相同，这时候我们需要给每一个云朵分别做一个动画，就会导致重复计算，代码冗余。有没有更好的方法呢？能不能让我只定义一个从`100vw`到`-10vw`的动画，让我在使用的时候指定其起始位置呢？

答案当然是有的，就是通过`animation-delay`属性的负值来实现。

## `animation-delay`负值的妙用

看到`animation-delay`时我们一眼就看出是延迟几秒执行，但是我们很少能够想到`animation-delay`还可以为负值。根据[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/animation-delay)上介绍，定义一个负值会让动画立即开始，但是动画会从它的动画序列中某个位置开始，例如如果设定值为-1s，动画会从的动画序列中的第1秒位置处立即开始，也可以这样理解，就是动画提前1s就开始执行，只是这1s的动画过程我们看不到，但是它的结果（位置偏移）确是发生了。

在`animation-delay`负值以后，代码如下：

```css
.cloud {
    animation: move-from-right-to-left 20s linear -10s infinite;
}

@keyframes move-from-right-to-left {
    0% { transform: translateX(100vw); }
    100% { transform: translateX(-10vw); }
}
```

这样一来，省去了繁琐的计算过程，只需要简单估算出`animation-delay`的数值即可，并且帧动画可以复用了。只要是从右往左移动的动画都可以，再配合`animation-direction`，从左往右的动画也可以复用，真是太方便了。

## 总结

这篇文章主要介绍了通过CSS3实现动画的两种方式`transition`和`animation`，简单介绍了`transition`的用法和局限，重点介绍了`animation`的用法，特别提示大家的是`animation-delay`可以是负值，并且有大用。

