---
title: CSS编码规范
---

为了提高 CSS 的可读性、维护性、扩展性、复用性，制定以下书写规范。

##1 命名规范

1. Id 和 Class 全部小写，并用中划线隔开。

##2 书写规范

1. 样式和内容分离，不使用 style 属性定义样式。
1. 【建议】属性书写顺序
   显示属性
   display, position, float
   盒模型属性
   width, max-width, min-width,
   height, max-height, min-height,
   border
   margin, margin-top, margin-right, margin-bottom, margin-left,
   padding, padding-top, padding-right, padding-bottom, padding-left,
   文本属性(color, font, text-decoration, text-align, vertical-align, white-space, other text, content)
1.
1. （由于性能问题）不许使用表达式`expression`，请使用 Javascript 替代。
1. 【建议】在模块的开头和结尾通过注释。

```css
/* XX模块 开始*/
/* XX模块 结束*/
```

http://blog.segmentfault.com/tychio/1190000000531277

CSS 命名规范
一、类名 !important
全部采用中划线命名法
<a rel="nofollow" href="javascript:void(0);" class="titleMore">More</a> // 错误
<a rel="nofollow" href="javascript:void(0);" class="title-more">More</a> // 正确
二、文件名 !important
CSS 文件名统一采用小写字母加下划线的方式，单词之间用中划线(-)分割, 不允许出现大写或者其他符号。正确的如：feedback-send.css
CSS 文件默认情况下的编码必须采用 utf-8（无 bom），其他编码方式均不可！！！（某些专门给中文用户使用的网页中的 CSS 除外）

书写规范
一. 不得在 css 中使用 expression !important
//由于性能的问题，不准使用 (通过重置触发该 Expression 的 CSS 属性来优化性能，但是也可能有其他问题)，请用 js 代替
.header {height:expression(document.documentElement.height)}

五. CSS 选择符组
如果是选择符组，则这些选择符(selector)各占一行
.style1,
.style2 {background:url(xxx.gif) no-repeat ;}

七. 采用属性缩写方式
.style {margin:10px 5px; font:normal 12px/1.5 Verdana; color:#F60; } //颜色值能简写的使用简写
属性缩写所代表的意思请点击 这里
七. 写成一行
.yellowish-btn,.yellow-btn{display:inline-block;zoom:1;border:0 none;background:none;}
.yellowish-btn:hover,.yellow-btn:hover {text-decoration:none;}
.yellowish-btn input,.yellow-btn input, .yellowish-btn button,.yellow-btn button {float:left;width:100%;border:0 none;}
.yellowish-btn input, .yellowish-btn button {height:27px;color:#7A2D01;}
为了方便区域阅读，属性写成一行。

框架设计原则

在使用中避免不必要的复杂
能用 CSS 完成的不动用其他资源手段

有原则的优雅退化
加速远古浏览器的消亡, 让现代浏览器更舒适

统一风格
让代码和使用方式以同一种机理对外服务

框架实现原则

科学的模块化, 减少不必要的依赖,降低耦合的可能性
以模块和模块组的概念提供积木, 不需要使用者做过多的额外编码工作
使用独立完整的代码片段, 结构化和规范化所有管辖范围内的编码方式
多组合, 少复写
HTML 结构设计原则

完整封闭的模块结构
语义化和可及性
杜绝不必要的冗余, 保留合理结构增强定义灵活性

命名规约

normalize(reset & typegraphy)直接以 HTML 元素作为选择器定义, 无额外 class 或 id 定义.
structure(grids & layouts)遵循定义中的命名方式, 无特殊要求.
utility 以 util-作为前缀, 标识为工具类, 无视觉样式定义.
mod 中各模块以 ui-作为前缀, 标识为与视觉样式相关模块.
以\_(下划线)作为前缀, 标识为内部未开放/不开放对外服务的部分.
模块名尽量让人看到名字就能知道是什么模块.

good case: ui-button

模块内部实现结构同样借鉴 YUI3 widget 文档中对于 widget 在结构上的规约, 模块同样存在 bounding box 和 content box, 但会根据模块本身的复杂度将这两层分层模型的规约进行合并, 分解或者省略, 而 bounding box 和 content box 指的都是 DOM 结构上的设计, 并非 CSS 的 class 声明.

bounding box

bounding box 是指模块最外层的包裹容器的 DOM 结构, 她更多时候承担的是申明独立模块边界的职责, 而她基本不会定义和视觉渲染样式相关的声明.

在 bounding box 上附着模块名 ui-{MOD_NAME}, 而 ui-{MOD_NAME}负责定义的包括: display, position, float
在 bounding box 上附着模块的矩阵(在下文中有详细描述)中的主维度状态 ui-{MOD_NAME}-{STATUS}, 例如-system, -customize
content box

content box 是定义具体模块内容的 DOM 结构, 她更多时候承担的是申明模块内部内容包裹的职责.

完整情况下, 在 content box 上附着 ui-{MOD_NAME}-content, 仅用以申明该结构承担具体模块内容
在 content box 上附着模块矩阵(在下文中有详细描述)中的次维度类型 ui-{MOD_NAME}-{TYPE}, 例如-error, -success, 她负责具体的样式声明
bounding box 和 content box 在模块实现中的具体操作说明

Q: 任何时候都需要定义 bounding box 和 content box 吗?

A:

bounding box 在任何情况下都需要定义, 因为 bounding box 标示模块的最外围容器.
content box 是否定义的情况就相对复杂了, 它是否需要声明, 或者是否需要独立声明是由模块在设计定义中区分出来的维度和一些认为判断性的原则决定的.
如果设计定义的分类能够在一个维度进行抽象, 那可以不显性声明 content box 的 class, 而 content box 的结构由 bounding box 的结构承担.
如果设计定义的分类是多维度的矩阵情况, 而设计定义作用的元素本身在语义结构合理性上判断不应由多层结构构成, 且能够通过复写的方式实现多维度定义的情况下, 可以不显性声明 content box 的 class, 而 content box 的结构由 bounding box 的结构承担.
如果设计定义的分类是多维度的矩阵情况, 在语义允许的情况下, 并且在使用复写的方式存在比较大的兼容性问题时, 那 content box 这层结构就必须存在, 为了让 bounding box 和 content box 上彼此附着的不同维度进行组合, 从而实现模块复杂多样性的矩阵.
就 bounding box 和 content box 在抽象过程中的示例:

多维度必须交叉组合实现定义的多样性
example: DPL 中对 feedback 的定义, 根据分析结果(如何分析设计定义就不展开了), 我们就能够建立一个简单的矩阵来描述设计定义中抽象出来的组合矩阵:

[ ['standalone','addon'],['alert','error','success'] ]

第一维度我们定义成: standalone(独立使用)和 addon(作为附属)

第二维度我们定义成: alert(警告), error(出错)和 success(成功)

ps. 两类维度并没有强制的规约, 仅从语义上描述分类的便利性上做出区分.

只有两个维度互相组合才能描述清楚模块在设计上的分类, 那在具体的 HTML 结构设计上, 就会变成这样:

<BLOCK class="ui-feedback ui-feedback-{STATUS}">
    <BLOCK class="ui-feedback-content ui-feedback-{TYPE}">
        …some contents here
    </BLOCK>
</BLOCK>
bounding box 上附着两种 class:

ui-feedback : MOD_NAME 部分, 仅声明当前这个闭合 DOM 文档结构是何种模块. 该声明对应的样式部分仅包含(如有需要): display, position, float 等和模块整体形式位置相关的定义.

ui-feedback-addon 或者 ui-feedback-standalone : MOD_NAME-{STATUS}部分, 作为多维度中模块的状态声明, 为多维度的划分方式提供第一级的 namespace.

content box 上附着两种 class:

ui-feedback-content : 与 ui-feedback 所起作用基本一致, 声明当前容器是 feedback 模块中具体内容的包裹容器, 不做样式定义声明用.

ui-feedback-error 或者 ui-feedback-alert 或者 ui-feedback-success : MOD_NAME-{TYPE}部分, 作为多维度中模块的类型声明, 为多维度的划分方式提供第二级的 namespace.

利用状态声明(bounding box 上)和类型声明(content box 上)来进行组合实现多重维度的设计定义:

ui-feedback-addon
|- ui-feedback-error
|- ui-feedback-alert
|- ui-feedback-success

ui-feedback-standalone
|- ui-feedback-error
|- ui-feedback-alert
|- ui-feedback-success
当然也能是:

ui-feedback-error
|- ui-feedback-addon
|- ui-feedback-standalone
...
多重维度可通过复写实现多样性
example: DPL 中对 button 的定义, 照例按照分析结果, 我们得到一个简单的矩阵来描述设计定义中抽象出来的组合矩阵:

[ [primary, normal] , [large, medium, small] ]

第一维度我们定义成: primary(主要的)和 normal(普通的)

第二维度我们定义成: large(大号), medium(中号)和 small(小号)

从定义分解上看, 也需要多重维度才能描述清楚模块的分类, 但是对于一个简单的 button 结构来说, 使用两层 DOM 来描述真的不靠谱且没意义. 再设计定义的维度分类之上我们在加入一个语义结构合理性的原则, 那由此, 具体的 HTML 结构设计就会是这样:

<INLINE class="ui-button ui-button-{STATUS} ui-button-{TYPE}" />

在这个例子中 bounding box 依旧坚挺的存在,而对于模型而言, content box 同样存在, 只是在实际情况下隐形了.

ps. 考虑到单一结构承担 bounding box 和 content box 时 class 值过长等人性判断介入的信息, 在实际声明中如果出现 content box 声明(注意, 不是 content box 模型)可有可无, 原则性选择不声明 content box 的 class.

单一维度
example: DPL 中对 balloon 的定义, 根据设计定义分类, balloon 模块只有一个维度需要描述--箭头出现的位置, 那在 HTML 结构上的设计是:

<BLOCK class="ui-balloon ui-balloon-{STATUS}">
    …some contents here
</BLOCK>
在这个例子中, bounding box 中的 STATUS 声明就足够将设计定义中的维度表达清楚了, 那content box的声明就没有为多维度构建可组合的职责了, 当然它可以继续充当其他职责, 比如(1)表达模块内部内容或(2)区分与内容无关的区块. 用balloon来举例, 在目前的实现中:

<div class="ui-balloon ui-balloon-tl">
    <div class="ui-balloon-content">
        balloon top left
    </div>
    <a class="ui-balloon-arrow"></a>
</div>
在该示例中同样存在ui-balloon-content的申明, 在结构上和上文中的ui-feedback-content很类似, 但是在balloon中更多承担的是区分在balloon内部内容和balloon的指向性箭头结构, 让其在语义化和操作便利性上更加合理.

对于某些特殊场景下需要的模块属性
example : DPL 中对 button 的定义, 还是用 button 来举例, button 除了在上文中具备的二维分类外, 还有一个特殊场景 disabled(禁用), 在模块多维度组合的抽象结构, disable 是和其他两个维度处于同等维度上, 但是在实现上基本可以用复写的方式完成, 以减少因为组合的维度增加导致代码组织上的大量重定义.

对于复写场景的编码技巧
这里阐述的是一种小技巧, 在出现.class-a.class-b 声明的场景时由于需要照顾到低级浏览器(shit!)的兼容性问题, 要想办法让在高级浏览器中能够工作的多 class 声明的方式同样在低级浏览器中同样生效, 运气好的话可以直接用 class-b 的声明完全复写成想要的实现, 但是遇到 class-b 同样也要单独服务或者需要和其他一个命名空间配合服务(比如.class-c.class-b)的时候, 这样的情况运气就用完了. 好吧, 只能在有限的条件下将 class-a 和 class-c 这两个命名空间区分开, 比方, 在.class-a.class-b 的前面加个 p...这只能小范围解决问题, 无法杜绝问题的发生, 认命吧, 骚年!

给一些命名后缀的建议
对于模块的分类维度, 在这里给出一些常见的命名建议, 省的大家在写模块时抓破头皮.

状态(STATUS)

system / customize
standalone / addon
horizontal / vertical
frontend / backend
wrap / separate
normal / primary
类型(TYPE)

error / alert / success
small / medium / large
block / nonblock
属性(ATTRIBUTE)

disabled / readonly
current / active
fixed / locked
