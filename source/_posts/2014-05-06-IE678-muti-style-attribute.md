---
title: IE6，7，8(Q)中同一元素重复定义的style属性会被合并
---

这个问题发生了有段时间了，现在把他记下来，一般人不会这么做，也不会遇到这种问题。但是你还是不能阻止有人不小心这么做了。出了问题就要解决，谁让咱是开发呢？

某次项目发布前，测试同事报出 Bug：产品编辑页面原本有的产品分组输入框不见了。

翻出我们的 VM 文件，代码中 #if 等是 velocity 语法。代码如下：

```html
<div
    id="productTeamSelect"
    class="ui-form-item"
    #if(!$webProductPolicy.canDisplayGroupName)style="display:none;"
    #end
></div>
```

既然是该 Div 不显示看了吗？那么我只需要打印下`$webProductPolicy.canDisplayGroupName`的值即可，该字段表示用户是否可以进行产品分组。如果其值是`false`，那么该用户不可以进行产品分组，就不显示。否则显示分组。通过打印发现其值为`false`，表示该用户不可以进行产品分组。那么不显示就是正常的。查看 Javascript 代码也发现：控制产品分组元素是否显示只出现在两个地方，但是都在 Click 事件回调中，并不会在加载页面之后就执行。所以不显示就是因为后端传来的值`$webProductPolicy.canDisplayGroupName`为`false`导致，交给后端同事处理。

很快后端同事给出了回复，他们没有修改任何有关于此的代码，并且提出线上`$webProductPolicy.canDisplayGroupName`的值也同样为`false`，但是线上却显示了产品分组元素，该产品已经下架，不能分组，也符合业务需求，所以后端认为当前线上显示了分组是错误的，而现在却是意外地改正了。

此时已经证明前端和后端都没有错误，而是当前线上代码有问题。线上代码是这样的：

```html
<div
    id="productTeamSelect"
    class="productTeamSelect"
    style="float:left;"
    #if(!$webProductPolicy.canDisplayGroupName)style="display:none;"
    #end
></div>
```

生成的页面源代码是：

```html
<div id="productTeamSelect" class="productTeamSelect" style="float:left;" style="display:none;"></div>
```

没有什么不对啊？`display`属性被设置为`none`，本不该显示的啊？为什么显示了呢？

只好通过`window.getComputedStyle`方法获取`display`属性值，看看浏览器是解析这段 HTML 的。实验发现`display`属性值竟然是`block`。奇怪了，为什么第二个 style 没有生效，难道是因为最近刚刚修改的 document type 造成的。然后我编写了两个测试页面，他们都有一个 DIV 元素，且都有两个 style 属性。唯一不同的是 document type，分别是 HTML5 的 document type：`<!DOCTYPE html>`，和旧的 document type 声明

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

测试发现，结果一致，后定义的 style 都没有生效。这个有点出乎我的意料，虽然之前没有看到规范中如何定义，但是潜意识中认为应该是两个 style 会整合在一起，并且在后定义覆盖先定义的。难道我的认识有误？

谷歌了一下，发现一篇文章：[IE6 IE7 IE8(Q) 中同一元素重复定义的 style 属性会被合并](http://www.w3help.org/zh-cn/causes/HY8002)，文章介绍了 IE6-8 的这个 Bug，并且指出“不要依赖 IE 的容错机制，避免重复定义 HTML 元素属性”。这个知识点比较冷，并且一般人都不会这么写，也不会遇到这个问题。现在回头看时才注意到通过 Firefox 产看源代码时`style="display:none;"`是被红色标注的。Firefox 意在提示我们第二个 style 属性没有生效，可惜这个重要信息被忽略了。

下架产品在 IE6，7 下不能分组，而在 IE8 和其他浏览器中却可以进行分组，为什么该代码的作者没有发现该问题呢，就不得而知了。难道该代码是很多年前的，那时候还没有 IE8+，Firefox 还不够流行，chrome 还没有诞生？

锁定根本原因就好办了，要么是继续保持网站内部逻辑，下架产品不能分组，即产品分组输入框不可见，修复线上故障。要么是取消以前的网站内部逻辑，下架产品也可以进行分组，这样后端输出的`$webProductPolicy.canDisplayGroupName`值改成`true`即可，最终效果保持和线上一致。最终由 PD 拍板决定采用后一个方法，修改网站内部逻辑，保持对最终用户的逻辑不变，下架产品也能进行分组。

以上 Bug 是解决了，但是我们应该吸取教训，经常梳理业务逻辑，经常审核代码。
