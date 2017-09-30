
1. ####请解释一下什么是语义化的HTML。
使用拥有语义的标签，比如h1-h6, article, section, aside, header, footer, nav, address, strong, em等标签，而不是不分清空皂白地统一使用div或者p标签。  
好处：（1）方便开发人员交流。统一的标签语义，不需要使用class表明用意。也阻止了不同的人使用不同的单词表达同样的含义。
（2）对搜索引擎友好。  
（3）样式加载失败时，浏览器默认样式也使得页面比较清晰。  
（4）对CSS支持不好甚至不支持的浏览器友好。  
  
1. ####你能描述一下渐进增强和优雅降级之间的不同吗?
构建站点的两个方法，都用到了特性检测。  
渐进增强：Progressive enhancement。先构建站点的最少特性，然后根据浏览器特性追加功能。  
优雅降级：graceful degradation。先构建站点的完整版本，然后根据浏览器特性进行修复。
  
1. ####为什么利用多个域名来存储网站资源会更有效？  
浏览器对同一个域的并行连接数是有限制的，当前一般浏览器4到8个，而IE6，7只有2个，如果同时下载同一个域下的很多资源，就会因为此限制而导致资源下载串行。但是使用多个域虽然可以并行下载资源，但是会使得浏览器需要解析更多的域名。所以在这对优缺点之间取一个平衡点。另外，解决此问题的办法并非只有启用多个域这一种方法，还可以合并和压缩JS和CSS，使用雪碧图等等。    
  
1. ####请说出三种减低页面加载时间的方法。（加载时间指感知的时间或者实际加载时间）
文件的合并以及压缩（减少了HTTP请求的数量）
使用CSS Sprites（减少HTTP请求的数量）
使用CDN托管（减少了数据传输时间）
使用CSS3，SVG，甚至Canvas替代图片。
有效地适度地使用缓存脚本和其他资源

1. ####什么是FOUC？你如何来避免FOUC？
FOUC全称是Flash Of Unstyled Content，翻译为文档样式闪烁。一般指IE在加载网页的时候，出现短暂的CSS样式失效。
问题一：什么时候出现呢？（1）IE浏览器（2）IE的临时文件夹没有缓存过页面的css文件（3）页面head标签里面没有任何link和script标签（4）页面引用样式的方式是采用@import（5）外在因素（比如网速等）
问题二：咋解决？在head里面加入一个link或者script标签
吐槽：我可能一辈子也不会写出这种代码，也希望面试官不要因为这种题目来拒绝候选人。

1. ####描述css reset的作用和用途。
浏览器都会有默认样式表，这样保证了页面中即便不写任何样式，浏览器仍然能够清晰地展示出页面内容，并突出重点。  

1. ####描述下float和它的工作原理。
float最初应用于图文混排，让文字环绕图片。  
语法为float: left/right/none;  
float box脱离当前文档流，定位到父元素或者另一个float box的边缘。父元素水平方向空间不足时，向下移动知道可以放下。文档流中inline box环绕float box，block box会和float box重叠，除非使用clear: both/left/right来清除浮动。float box会形成一个新的block formatting context（BFC块状格式化上下文）。float box不会超出所在BFC，也不会和其他BFC重叠。

1. ####清除浮动的方法有那些，分别适用于什么情形。
clear: both/left/right  
用于清除相应元素之前的兄弟节点浮动的影响。left和right指浮动方向，both包含left和right两个方向。

1. ####解释css sprites，如何使用。
将数张小的背景图片拼合成一张大的图片，然后在CSS中结合background-image，background-position，width和height等属性显示某张小的背景图片。  
网上有不少小工具可以让你上传一些小图片，然后生成一个大图并生成相应的样式表。即可直接使用。  
优点：减少了图片的请求数量。  
缺点：（1）不适合需要repeat的背景图片。（2）需要额外的工作来拼合成大图。（3）CSS和图片缓存失效的概率和影响更大。  
1. ####你最喜欢的图片替换方法是什么，你如何选择使用。
Web-Font, CSS3, SVG, Canvas  
关于如何选择，首先查看所要兼容的浏览器是否兼容其特性。在三者均支持的很好的情况下，选择优先级从高到低分别是Web-Font, CSS3, SVG, Canvas  
Web-Font适合单色图片，并且可以通过font属性来设置其大小，颜色等，特别适合用于响应式设计中。
CSS3基本可以说是从SVG发展而来的，功能应该还不如SVG，但是其足够简单，所以选择优先级是比较高的。
SVG适合需要响应用户行为，比如mousemove等操作的图片，作为矢量图，适合用于响应式设计中。  
Canvas适合显示动态生成的图表，不需要响应用户行为。  

1. ####讨论CSS hacks，条件引用或者其他。 

1. ####解释BFC（block formatting context，块状布局上下文）
把一组块级box和浮动元素放在一起布局的区域。  
特点：（1）不折叠边距（margin collapsing），在同一个BFC是折叠边距的其中一个必要条件。（2）浮动元素不会超出它的范围。（3）不会跟其他浮动元素重叠。  
形成新的BFC的条件：（1）float属性不为none（2）overflow属性不为visible（3）display属性是table-cell、table-caption或inline-block（4）position属性不为static或relative。
IE6/7没有BFC概念，但是它们有hasLayout与之相似，可以通过zoom: 1触发。