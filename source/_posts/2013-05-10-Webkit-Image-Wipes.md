---
title: 【翻译】Webkit图片擦拭效果
---

[来源](http://css-tricks.com/webkit-image-wipes/)  
Webkit浏览器支持遮罩，虽然它还不是标准。就像Photoshop中一样，你声明一张图片作为遮罩，黑色部分是不透明的，将会遮住其后面的元素。白色部分透明，其后面的元素是可见的。灰色部分是半透明的。所以下面的图片：
```html
<img src="orig.jpg" alt="trees" class="circle-mask">
```
![orig.jpg](http://cdn.css-tricks.com/wp-content/uploads/2010/12/orig.jpg)
以及遮罩图片：
![mask.png](http://cdn.css-tricks.com/wp-content/uploads/2010/12/mask.png)
应用如下CSS：
```css
.circle-mask {
  -webkit-mask-box-image: url(mask.png);
}
```
将会得到如下效果：
![masked.png](http://cdn.css-tricks.com/wp-content/uploads/2010/12/masked.png)

###遮罩并不一定要真正的图片
这里用到的第一个技巧是，声明为遮罩`webkit-mask-box-image`的图片并不是真正的图片，而是使用`-webkit-gradient`来实现。当然我们可以使用渐变创建一张图片，但是通过编程创建渐变遮罩更容易修改，并且减少一次HTTP请求。  
```css
-webkit-mask-position: 0 0;
-webkit-mask-size: 200px 200px;
-webkit-mask-image: -webkit-gradient(linear, left top, right bottom, 
   color-stop(0.00,  rgba(0,0,0,1)),
   color-stop(0.45,  rgba(0,0,0,1)),
   color-stop(0.50,  rgba(0,0,0,0)),
   color-stop(0.55,  rgba(0,0,0,0)),
   color-stop(1.00,  rgba(0,0,0,0)));
```
在以上的CSS中，我们创建了一个200X200像素的图片，顺着45度角方向，从左上角的完全不透明，到差不多一半的地方渐变到完全透明。就像下面的图片：
![diagonalgradientmask.png](http://cdn.css-tricks.com/wp-content/uploads/2010/12/diagonalgradientmask.png)
###移动遮罩
在上面我们通过`-webkit-mask-position`设置遮罩的位置，因为可以设置其位置，所以也就可以移动它。我们可以在`:hover`伪类上移动，
```css
.circle-mask {
  -webkit-mask-position: 0 0;
}
.circle-mask:hover {
  -webkit-mask-position: -300px -300px;
}
```
也可以使用`-webkit-sanimation`来自动移动遮罩。
```css
@-webkit-keyframes wipe {
  0% {
    -webkit-mask-position: 0 0;
  }
  100% {
    -webkit-mask-position: -300px -300px;
  }
}
.circle-mask {
  -webkit-animation: wipe 6s infinite;
  -webkit-animation-delay: 3s;
  -webkit-animation-direction: alternate;
}
```
###创建擦拭效果
![](http://cdn.css-tricks.com/wp-content/uploads/2010/12/wipe.jpg)
我相信聪明的你已经将他们联系在一起。这个主意就是将一张图片放在另一张图片上面，上面的图片作为遮罩，根据需要移动遮罩。  
``` html
<div id="banner">
  <div><img src="images/banner-1.jpg" alt="Skyline 1"></div>
  <div><img src="images/banner-2.jpg" alt="Skyline 2"></div>
</div>
```
```css
#banner {
  width: 800px;        /* Size of images, will collapse without */
  height: 300px;
  position: relative;  /* For abs. positioning inside */
  border: 8px solid #eee;
  -webkit-box-shadow: 1px 1px 3px rgba(0,0,0,0.75);
}

#banner div {
  position: absolute; /* Top and left zero are implied */
}

/* Second one is on top */
#banner div:nth-child(2) {
  -webkit-animation: wipe 6s infinite;
  -webkit-animation-delay: 3s;
  -webkit-animation-direction: alternate;
  -webkit-mask-size: 2000px 2000px;
  -webkit-mask-image: -webkit-gradient(linear, left top, right bottom, 
      color-stop(0.00,  rgba(0,0,0,1)),
      color-stop(0.45,  rgba(0,0,0,1)),
      color-stop(0.50,  rgba(0,0,0,0)),
      color-stop(0.55,  rgba(0,0,0,0)),
      color-stop(1.00,  rgba(0,0,0,0)));
}
```

###示例和下载
在下载文件中，有另一个例子，水平方向上擦拭而不是顺着某个角度，并且使用`-webkit-transition`而不是动画。  
[查看例子](http://css-tricks.com/examples/ImageWipes/) 
[下载](http://css-tricks.com/examples/ImageWipes.zip)

###比两个更多？
我花了更多时间在尝试能否连续擦拭三张照片。这是可能的但是我不能让它足够平滑，以及像我想象的那么方便，所以我放弃了。我仍然非常确信这是可以的，也许使用两个不同的有着不同延迟的动画。如果你尝试了并成功了，一定要给我看看。

###更多
要了解更多请查看webkit在2008年的一个关于遮罩的[通告](http://webkit.org/blog/181/css-masks/)，那里有很多有用的信息，比如遮罩图片可以伸展（像[full page backgrounds](http://css-tricks.com/perfect-full-page-background-image/)）以及重复。实际上他的作用和带有九宫格[border-image](http://css-tricks.com/understanding-border-image/)有很大相似之处，

###Credit
我从[Doug Neiner](http://dougneiner.com/)那里偷来了这个想法，他给我展示了一些关于这些想法的例子。经他允许发此文章。