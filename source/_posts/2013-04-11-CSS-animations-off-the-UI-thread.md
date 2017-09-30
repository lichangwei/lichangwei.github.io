---
title: 脱离UI线程的CSS3动画
---

【原文】[CSS animations off the UI thread](http://www.phpied.com/css-animations-off-the-ui-thread/)  
我们知道浏览器是单线程的，在执行js脚本期间，UI会被冻结，用户行为也得不到响应。也就是所谓的“假死”现象。  
在最新版本的浏览器中，开启了一种新的实现，将CSS3动画移出UI线程。
## 测试结果 
* Win7上Chrome25支持
* iOS6上Chrome和Safari支持
* 原文中提到IE10（估计是Win8上），Firefox OS支持。  

## 测试方法
[点击我打开测试页面](http://www.phpied.com/files/css-thread/thread.html)。点击按钮以后，javascript脚本持续执行2秒，在这个过程中查看红蓝绿三个方块的效果。如果三个均停止动画，说明浏览器尚不支持将CSS transform动画移出UI线程，如果红色和绿色方块继续动画，则支持。
## 解释
红色方块的动画如下
```css
.spin {
  animation: 3s rotate linear infinite;
}
@keyframes rotate {
  from { transform: rotate(0deg);  }
  to   { transform: rotate(360deg);}
}
```
绿色方块的动画如下
```css
.walkabout-new-school {
  animation: 3s slide-transform linear infinite;
}
@keyframes slide-transform {
  from { transform: translatex(0);    }
  50%  { transform: translatex(300px);}
  to   { transform: translatex(0);    }
}
```
蓝色方块的动画如下
```css
.walkabout-old-school {
  animation: 3s slide-margin linear infinite;
}
@keyframes slide-margin {
  from { margin-left: 0;   }
  50%  { margin-left: 100%;}
  to   { margin-left: 0;   }
}
```

## 结论
如果有可能，尽量使用CSS3 transform动画。这些动画脱离了主线程，不影响主线程。