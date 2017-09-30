---
title: 使用别样的鼠标形状
---

某些时候，我们并不使用CSS中支持的鼠标形状，比如default箭头，pointer手型等等，也不使用url指定鼠标形状，而是使用html+css+js渲染。例如在黑色背景的页面中，使用一个从圆心到周边透明度径向渐变的圆形图形作为鼠标形状。
实现起来非常简单。首先在html元素上声明如下样式`html{ cursor: none; }`，放弃默认的箭头鼠标形状。然后创建一个绝对定位的，className为cursor的body子元素div。
## 方案一：渐变背景色实现【失败】
```css
div.cursor{
  position: absolute;
  width: 150px;
  height: 150px;
  margin-top: -75px;
  margin-left: -75px;
  background: -webkit-radial-gradient(50% 50%, circle contain, rgba(255, 255, 255, 1.0), rgba(255, 255, 255, 0));
}
```
但是这样就有一个问题，当按下鼠标时，mouse事件的target属性永远指向该DIV元素。如果想监听页面上其他元素的mouse事件，就变得很繁琐，只能根据。于是首先想到的是用**after伪元素**实现。
## 方案二：伪元素背景色实现【失败】

```css
div{
  position: absolute;
  width: 0;
  height: 0;
}
div:after{
  position: absolute;
  content: '';
  width: 150px;
  height: 150px;
  margin-top: -75px;
  margin-left: -75px;
  background: -webkit-radial-gradient(50% 50%, circle contain, rgba(255, 255, 255, 1.0), rgba(255, 255, 255, 0));
}
```
实现以后发现mouse事件的target属性仍然指向该div元素。思考一下，这个是符合逻辑的，因为first和after伪元素本来就相当于子元素的。
## 方案三：box-shadow效果实现【成功】
也许你还记得，很多阴影效果都有模糊渐变的效果，在box-shadow语法中，blur配合spread就可以做到。
语法：`box-shadow: h-shadow v-shadow blur spread color inset;`
```css
div{
  position: absolute;
  width: 0;
  height: 0;
  -webkit-box-shadow: 0 0 75px 30px #000;
}
```
效果基本可以实现，但是对于更加特别的需求，就怕无能为力了。
## 方案四：CSS属性pointer-events使得元素不响应事件【最佳】
事件可以透过（忽视）pointer-events值为none的任何元素，而触发在它后面的那个元素之上。
```css
pointer-events: none;
```
