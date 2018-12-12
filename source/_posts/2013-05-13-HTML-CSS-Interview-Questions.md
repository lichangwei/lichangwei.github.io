---
title: HTML & CSS 面试题
sage: true
---

1. ####请解释一下什么是语义化的 HTML。
   使用拥有语义的标签，比如 h1-h6, article, section, aside, header, footer, nav, address, strong, em 等标签，而不是不分清空皂白地统一使用 div 或者 p 标签。  
   好处：（1）方便开发人员交流。统一的标签语义，不需要使用 class 表明用意。也阻止了不同的人使用不同的单词表达同样的含义。
   （2）对搜索引擎友好。  
   （3）样式加载失败时，浏览器默认样式也使得页面比较清晰。  
   （4）对 CSS 支持不好甚至不支持的浏览器友好。

1. ####你能描述一下渐进增强和优雅降级之间的不同吗?
   构建站点的两个方法，都用到了特性检测。  
   渐进增强：Progressive enhancement。先构建站点的最少特性，然后根据浏览器特性追加功能。  
   优雅降级：graceful degradation。先构建站点的完整版本，然后根据浏览器特性进行修复。

1. ####为什么利用多个域名来存储网站资源会更有效？  
   浏览器对同一个域的并行连接数是有限制的，当前一般浏览器 4 到 8 个，而 IE6，7 只有 2 个，如果同时下载同一个域下的很多资源，就会因为此限制而导致资源下载串行。但是使用多个域虽然可以并行下载资源，但是会使得浏览器需要解析更多的域名。所以在这对优缺点之间取一个平衡点。另外，解决此问题的办法并非只有启用多个域这一种方法，还可以合并和压缩 JS 和 CSS，使用雪碧图等等。

1. ####请说出三种减低页面加载时间的方法。（加载时间指感知的时间或者实际加载时间）
   文件的合并以及压缩（减少了 HTTP 请求的数量）
   使用 CSS Sprites（减少 HTTP 请求的数量）
   使用 CDN 托管（减少了数据传输时间）
   使用 CSS3，SVG，甚至 Canvas 替代图片。
   有效地适度地使用缓存脚本和其他资源

1. ####什么是 FOUC？你如何来避免 FOUC？
   FOUC 全称是 Flash Of Unstyled Content，翻译为文档样式闪烁。一般指 IE 在加载网页的时候，出现短暂的 CSS 样式失效。
   问题一：什么时候出现呢？（1）IE 浏览器（2）IE 的临时文件夹没有缓存过页面的 css 文件（3）页面 head 标签里面没有任何 link 和 script 标签（4）页面引用样式的方式是采用@import（5）外在因素（比如网速等）
   问题二：咋解决？在 head 里面加入一个 link 或者 script 标签
   吐槽：我可能一辈子也不会写出这种代码，也希望面试官不要因为这种题目来拒绝候选人。

1. ####描述 css reset 的作用和用途。
   浏览器都会有默认样式表，这样保证了页面中即便不写任何样式，浏览器仍然能够清晰地展示出页面内容，并突出重点。

1. ####描述下 float 和它的工作原理。
   float 最初应用于图文混排，让文字环绕图片。  
   语法为 float: left/right/none;  
   float box 脱离当前文档流，定位到父元素或者另一个 float box 的边缘。父元素水平方向空间不足时，向下移动知道可以放下。文档流中 inline box 环绕 float box，block box 会和 float box 重叠，除非使用 clear: both/left/right 来清除浮动。float box 会形成一个新的 block formatting context（BFC 块状格式化上下文）。float box 不会超出所在 BFC，也不会和其他 BFC 重叠。

1. ####清除浮动的方法有那些，分别适用于什么情形。
   clear: both/left/right  
   用于清除相应元素之前的兄弟节点浮动的影响。left 和 right 指浮动方向，both 包含 left 和 right 两个方向。

1. ####解释 css sprites，如何使用。
   将数张小的背景图片拼合成一张大的图片，然后在 CSS 中结合 background-image，background-position，width 和 height 等属性显示某张小的背景图片。  
   网上有不少小工具可以让你上传一些小图片，然后生成一个大图并生成相应的样式表。即可直接使用。  
   优点：减少了图片的请求数量。  
   缺点：（1）不适合需要 repeat 的背景图片。（2）需要额外的工作来拼合成大图。（3）CSS 和图片缓存失效的概率和影响更大。
1. ####你最喜欢的图片替换方法是什么，你如何选择使用。
   Web-Font, CSS3, SVG, Canvas  
   关于如何选择，首先查看所要兼容的浏览器是否兼容其特性。在三者均支持的很好的情况下，选择优先级从高到低分别是 Web-Font, CSS3, SVG, Canvas  
   Web-Font 适合单色图片，并且可以通过 font 属性来设置其大小，颜色等，特别适合用于响应式设计中。
   CSS3 基本可以说是从 SVG 发展而来的，功能应该还不如 SVG，但是其足够简单，所以选择优先级是比较高的。
   SVG 适合需要响应用户行为，比如 mousemove 等操作的图片，作为矢量图，适合用于响应式设计中。  
   Canvas 适合显示动态生成的图表，不需要响应用户行为。

1. ####讨论 CSS hacks，条件引用或者其他。

1. ####解释 BFC（block formatting context，块状布局上下文）
   把一组块级 box 和浮动元素放在一起布局的区域。  
   特点：（1）不折叠边距（margin collapsing），在同一个 BFC 是折叠边距的其中一个必要条件。（2）浮动元素不会超出它的范围。（3）不会跟其他浮动元素重叠。  
   形成新的 BFC 的条件：（1）float 属性不为 none（2）overflow 属性不为 visible（3）display 属性是 table-cell、table-caption 或 inline-block（4）position 属性不为 static 或 relative。
   IE6/7 没有 BFC 概念，但是它们有 hasLayout 与之相似，可以通过 zoom: 1 触发。
