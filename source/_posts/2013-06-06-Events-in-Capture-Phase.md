---
title: 通过body的error事件捕获页面中所有图片的error事件
---

在BDP项目中，根据用户输入的关键字搜索视频并显示出来，后端返回的每条记录中有一个数组字段`image_urls`表示该视频的缩略图地址，它共有三种类型，分别是small, medium和large。
```json
{
  "image_urls": {
    "small": "http://...",
    "medium": "http://...",
    "large": "http://..."
  }
}
```
需求是：在显示该条记录时，如果有small，就显示small，没有small但是有medium，就显示medium，如果没有small和medium但是有large，就显示large。如果三者都没有，那么显示一个默认的图片。实现起来非常简单。如下：
```js
var default_icon = '/images/default_icon.png';
item.icon = image_urls.small || image_urls.medium || image_urls.large || default_icon;
```
这个时候已经隐隐觉得不妥，因为给该缩略图的准备的位置只有84*84px。也就是说
```css
.icon {
  max-width: 84px;
  max-height: 84px;
}
```
如果某条记录只有large缩略图，并且尺寸远远大于84*84px，那么非常浪费带宽，对于性能很差的蓝光机浏览器来说，浏览器渲染若干张那么大的图片，使得性能更加低下。通知需求方以后，他们回复是这些记录有可能是从第三方服务获取，后端不可能准备合适尺寸的缩略图，但是他们又想尽可能显示缩略图，所以只能这样。  

还有另外一个问题：当一条记录有small和medium缩略图，但是small缩略图地址是无效地址，此时并不会显示medium缩略图。跟需求方沟通这种情况能否接受，但是需求方没有妥协，他们希望越完善越好。希望后端不要返回无效的地址，因为这样第一做起来比较复杂，第二这样可能无故增加很多无效的404请求。但是后端的回复仍然是有些记录来自第三方服务，后端不会判断缩略图地址是否有效。所以这时候的需求就变成：
在显示某条记录时，如果有small地址有效，那么显示small，如果small地址无效，但是medium地址有效，那么显示medium，如果small和medium地址都无效，但是large地址有效，那么显示large，如果三者都无效，那么显示默认图片。  

由于img的error事件并不会冒泡，只能分别处理，要在所有的img元素上都监听error事件真是令人头大。所幸我们使用基于字符串的模板引擎，onerror方法只需写在模板中即可。实现如下。
```html
<!--use doT as template engine-->
<img src="{{=item._src}}" images="{{=item._images}}" imageIndex="1" 
  onerror="var images=this.getAttribute('images').split('|');var index=parseInt(this.getAttribute('imageIndex'),10);if(index<images.length){this.src=images[index];this.setAttribute('imageIndex', index+1)}"/>
```
```js
var images = [];
var urls = item.image_urls;
if(urls){
  if(urls.small    ) images.push(urls.small    );
  if(urls.medium   ) images.push(urls.medium   );
  if(urls.thumbnail) images.push(urls.thumbnail);
}
images.push(default_thumb);
item._images = images.join('|');
item._src = images[0];
```
就像我们想象中的一样，解决了问题。但是在《Javascript权威指南（第六版）》中17.2.2节中提到，以上的做法有着坏味道，第一点，将html视图和js控制逻辑混和在一起；第二点，当指定一个字符串作为事件处理程序时，浏览器会做以下事情：
```js
function(event) {
  with(document) {
    with(this.form || {}) {
      with(this) {
        /* your code here */
      }
    }
  }
}
```
这样通过`onXxx=""`属性绑定事件处理程序的方法是每一个热爱代码热爱前端的人所不能忍受的。  

后来持续地在网上寻找更好的解决方案，终于老天开眼，找到下面这篇文章[Error events don’t bubble from images and how to work around that](http://m.cg/post/30934181934/error-events-dont-bubble-from-images-and-how-to-work)，文章中指出在W3C的DOM Level 2规范中指定[error事件应该冒泡](http://www.w3.org/TR/DOM-Level-2-Events/events.html#Events-eventgroupings-htmlevents)，但是只有Opera实现了这点。应该是由于其他的浏览器的反对，最后在DOM Level 3规范中[取消的error事件的冒泡](http://www.w3.org/TR/DOM-Level-3-Events/#event-type-error)。作者还指出一种解决办法就是：使用body元素的捕获处理程序。
```js
document.body.addEventListener('error', handleResourceError, true);
```
顺便提及一下，作者最后提出`<body onerror="handleResourceError()">`相当于在window上注册了事件并可以监听到脚本（运行时）错误。这个论点在Chrome27 **没有** 得到支持。  

所以最后的比较完美的解决方案是：
```html
<img src="{{=item._src}}" images="{{=item._images}}" imageIndex="1">
```
```js
// 部分代码同上，略去
document.body.addEventListener('error', function(e){
  var target = e.target
  if(!target || target.tagName !== 'IMG') return;
  var images = target.getAttribute('images').split('|');
  var index = parseInt(target.getAttribute('imageIndex'), 10);
  if(index < images.length){
    target.src = images[index];
    target.setAttribute('imageIndex', index+1);
  }
}, true);
```
在《Javascript权威指南（第六版）》中17.3.6节中提到：事件传播的捕获阶段就像反向的冒泡阶段，一次调用window，document，body...直至父节点之上的捕获处理程序，在目标对象绑定的捕获事件处理程序不会被调用。事件捕获提供了在事件还没有传播到目标节点之前查看他们的机会，可以用于调试，或者过滤事件。比如用于拖放，拖放的处理通常不是鼠标点击的目标元素。  
