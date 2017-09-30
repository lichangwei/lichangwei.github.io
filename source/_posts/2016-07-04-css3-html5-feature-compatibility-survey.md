---
title: 主要CSS3，HTML5特性兼容性调查
---

## CSS3 HTML5 部分特性

|特性组|特性|IE8|IE9|IE10|IE11|Edge|CH|FF|SF|参考|
|---|:----|:----:|:----:|:----:|:----:|:----:|:----:|:----|
|values|
||calc|×|√|√|√|√|√|√|√|[caniuse](http://caniuse.com/#search=calc)|
||rem|×|√|√|√|√|√|√|√|[caniuse](http://caniuse.com/#search=rem)|
||vw,vh|×|√|√|√|√|√|√|√|[caniuse](http://caniuse.com/#search=vw)|
|color|
||currentColor|×|√|√|√|√|√|√|√||
||hsl hsla rgba|×|√|√|√|√|√|√|√||
|background|
||border-radius|×|√|√|√|√|√|√|√||
||border-images|×|×|×|√|√|√|√|√||
|images|
||linear-gradient|×|×|√|√|√|√|√|√||
||radial-gradient|×|×|√|√|√|√|√|√||
||repeating-linear-gradient|×|×|√|√|√|√|√|√||
||repeating-radial-gradient|×|×|√|√|√|√|√|√||
|animation|
||keyframes|×|×|√|√|√|√|√|√||
||requestAnimationFrame|×|×|√|√|√|√|√|√||
|transform|
||2d|×|√|√|√|√|√|√|√||
||3d|×|×|√|√|√|√|√|√||
|postion|
||position:sticky|×|-|-|-|-|-|√|√|[caniuse](http://caniuse.com/#search=sticky), [polyfill](https://github.com/wilddeer/stickyfill)|
|selector|
||querySelector querySelectorAll|√|√|√|√|√|√|√|√||
|webfonts|
||@font-face|√|√|√|√|√|√|√|√|[小米案例](http://www.mi.com/mi4/)|
|ui|
||sizing-box|√|√|√|√|√|√|√|√||
|dom|
||mutation-observers|×|-|-|√|√|√|√|√|[caniuse](http://caniuse.com/#search=MutationEvents), [polyfill](https://github.com/webmodules/mutation-observer)|
|HTML5|
||audio|×|√|√|√|√|√|√|√||
||video|×|√|√|√|√|√|√|√||
|ECMA5|
||Promises|-|-|-|-|√|√|√|√||

## CSS选择器

|-|CSS选择器|IE8+|IE9|IE10+|
|---|
|CSS2.1|
|\*|通配符|√|√|√|
|#foo|ID选择器|√|√|√|
|.foo|类选择器|√|√|√|
|foo|节点选择器|√|√|√|
|\>|子选择器|√|√|√|
|+|后一个兄弟节点|√|√|√|
|[attr]|带有attr属性的元素|√|√|√|
|[attr="val"]|&lt;div attr="val"/&gt;|√|√|√|
|[attr~="val"]|&lt;div attr="val val1"/&gt;|√|√|√|
|[attr&#124;="val"]|&lt;div attr="val-val1"/&gt;|√|√|√|
|:first-child|第一个子元素|√|√|√|
|:link, :visited|伪类，仅适用于a标签|√|√|√|
|:active, :hover|伪类，适用于所有元素|√|√|√|
|:focus|伪类，获取焦点的元素|√|√|√|
|:lang()|伪类，&lt;div lang="en"/&gt;|√|√|√|
|CSS3|
|[attr^="val"]|带有attr属性并且值以val打头|√|√|√|
|[attr$="val"]|带有attr属性并且值以val结束|√|√|√|
|[attr*="val"]|带有attr属性并且值中带有val字眼|√|√|√|
|~|后面兄弟选择器|√|√|√|
|:last-child|最后一个子元素|×|√|√|
|:root|根元素|×|√|√|
|:only-child|唯一的子节点|×|√|√|
|:nth-child()|选择一个或多个子元素，参数n代表0-+∞|×|√|√|
|:nth-last-child()|选择一个或多个子元素，参数n代表0-+∞，从后往前选择|×|√|√|
|tag:only-of-type|唯一tag元素|×|√|√|
|tag:nth-of-type|选择一个或多个子tag元素，参数n代表0-+∞|×|√|√|
|tag:nth-last-of-type()|选择一个或多个子tag元素，参数n代表0-+∞，从后往前选择|×|√|√|
|tag:first-of-type|选择第一个tag元素|×|√|√|
|tag:last-of-type|选择最后一个tag元素|×|√|√|
|:empty|匹配空节点|×|√|√|
|:target|匹配hash相应的元素|×|√|√|
|:enabled, :disabled|匹配表单中启用或禁用的元素|×|√|√|
|:checked|匹配表单中选中的元素|×|√|√|
|:not()|否定选择器|×|√|√|
备注：没有添加伪元素。

## CSS2.1之前的一些不为人知的特性

|特性|可选值以及含义|
|---|:----|
|empty-cells|hide：当td的子元素都是隐藏的时候，自动隐藏该td的边框和背景<br/>show：永远显示<br/>不过该属性只有在border-collapse:separate;时才有效。|


## 参考：
1. [CSS Current Work](http://www.w3.org/Style/CSS/current-work) 这里记载着CSS规范子集的进度（稳定级别）：工作草案，最后通告，候选标准，推荐标准和标准。
2. [W3C Technical Report Development Process](http://www.w3.org/2005/10/Process-20051014/tr.html) W3C技术报告开发过程
3. [《W3C技术报告开发过程》](http://www.w3ctech.com/topic/746) 只翻译一部分
4. [Mozilla CSS support chart](https://developer.mozilla.org/en-US/docs/Web/CSS/Mozilla_support_chart) 火狐浏览器CSS特性支持
5. [Safari CSS Reference](https://developer.apple.com/library/safari/documentation/AppleApplications/Reference/SafariCSSRef/Introduction.html)
6. [Comparison of layout engines (Cascading Style Sheets)](https://en.wikipedia.org/wiki/Comparison_of_layout_engines_(Cascading_Style_Sheets)) 中文版[浏览器引擎CSS支持比较](https://zh.wikipedia.org/zh-cn/%E7%80%8F%E8%A6%BD%E5%99%A8%E5%BC%95%E6%93%8ECSS%E6%94%AF%E6%8F%B4%E6%AF%94%E8%BC%83)
4. http://t337.org-w3c-region-chinese-html5.w3ctalk.info/w3c-t337.html