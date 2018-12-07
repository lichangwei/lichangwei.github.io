---
title: CSS3 Transition过渡属性探索
---

## 背景介绍

在前面的文章[蓝光机 WebApp-无尽列表优化](./2013-04-07-Blu-ray-Web-App-Endless-List-Optimization.md)中提到无尽列表，通过上下键可以查看前两个或者后两个记录。这时候需要做出动画效果。

## 解决问题

这里首先要说明 Opera 的一个问题，Opera 从 9.8 版本以后就定格为 9.8，如果要查看其真正版本，只有通过其 UserAgent 中‘Version/12.11’类似的文本来确定。在[Oprea 的怪异识别码](http://blog.imbolo.com/oprea-version/)说明了这一问题的由来。

> Opera 的工程师在测试中发现，Opera 10 Alpha 在很多老网站上的运作很不正常。原来，有很多网站使用了“浏览器嗅探”技术，也就是说网站会针对不同的浏览器提供不同的内容或功能。然而不幸的是，这些网站无法识别两位数的浏览器版本号，于是 Opera 就成了首当其冲的受害者——它们把 Opera 10.0 误认为是 Opera 1.0，并因此向 Opera 10 提供不完整的功能，甚至有些网站干脆拒绝 Opera 10 的访问。
> 这当然是 Opera 不愿意看到的。于是，Opera 的工程师们决定，将用户代理信息中的版本号定格在 9.8，并另外启用 Version 字段来标识真正的版本号。当然他们也考虑过选用 9.99 这个最接近 10 的可用作版本号的数字，不过出于预留空间的考虑，最终还是决定采用 9.8 作为象征性的版本号——它正好介于（Opera 9 的最后一个版本号）9.6 与 10 之间。

正是因为 Opera 这么委曲求全的决策，所以一直以为蓝光机中是 Opera9.8。而很多 CSS3 特性都不被 Opera9.8 支持。包括我们这里将会被用到的 CSS3 Transition。所以很自然地使用 jQuery 提供的 animate 方法实现。

```js
// top from 0 to 94
$list.animate({ top: top }, 'fast', function() {
    // do some thing
});
```

在蓝光机中该动画只会执行三步，分别是 top = 0, ~59, 94px，所以会给人一种很不平滑的感觉。

很长时间以后才注意到蓝光机中的浏览器是 Opera12.11. 然后开始尝试使用 CSS3 Transition。

```css
transition: top 300ms linear;
```

```js
$list.css('top', top);
```

以上动画效果在桌面浏览器上运行的非常好，但是在蓝光机上依然没有明显改善，反而感觉有点晃动。优化失败。

在尝试将其他部分动画也使用 CSS3 Transition 时发现一个问题，就是修改某个属性之前，如果该属性没有显式声明，那么各个浏览器处理方式不同。以 width 为例，这时通过`elem.style.width`得到一个空字符串。Chrome26 中，width 会从 0 过渡到 XX px，而在 Opera12 以及 Firefox20 中 width 直接变成 XX px，并且不会触发 transitionend 事件。这个小问题让我耽误了不少时间。

###使用方法

> transition ： [<'transition-property'> || <'transition-duration'> || <'transition-timing-function'> || <'transition-delay'> [, [<'transition-property'> || <'transition-duration'> || <'transition-timing-function'> || <'transition-delay'>]]_
> 或者
> transition-property ： none | all | [ <IDENT> ][ ',' <ident> ]_；  
> transition-duration ： <time> [, <time>]_  
> transition-timing-function ： ease | linear | ease-in | ease-out | ease-in-out | cubic-bezier(<number>, <number>, <number>, <number>) [, ease | linear | ease-in | ease-out | ease-in-out | cubic-bezier(<number>, <number>, <number>, <number>)]_  
> transition-delay ： <time> [, <time>]\*

transition-property 支持的属性参见[W3C 标准](http://www.w3.org/TR/css3-transitions/#transition-property)。  
transition-duration 和 transition-delay 支持如`1s`，`200ms`的值。

###总结

1. CSS3 Transition 可以在设置 CSS 属性时使其效果平滑过渡，但是过渡过程中，通过`elem.style.attr`获取其相应属性值时，得到的都是其最终值，这点和 jQuery 实现的动画效果不同。
1. 如果属性值为显式声明时，各个浏览器处理方式不同。以 width 为例，这时通过`elem.style.width`得到一个空字符串。Chrome26 中，width 会从 0 过渡到 XX px，而在 Opera12 以及 Firefox20 中 width 直接变成 XX px，并且不会触发 transitionend 事件。
1. Opera 在 9.8 版本以后就将版本定格为 9.8，在以后的 Opera 版本中只能通过其 UserAgent 中的'Version/12.11'之类的字符串来确定其版本号。
1. 如果设置的属性值跟设置前一样，那么并不会触发 transitionend。这点需要特别注意。
