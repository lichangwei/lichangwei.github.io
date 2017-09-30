---
title: 在DOM操作中，如何减少reflow或repaint
---

在这篇博客中，我们列举一些可以减少reflow或者repaint的方法。下面先讲述一下这两个概念。  
**reflow**，翻译为重排，意为重新布局。当一个元素的形状位置发生改变时，浏览器会对该元素以及子元素（也可能包括同级元素）进行布局和重绘。称之为重排。  
**repaint**，翻译为重绘。当一个元素的颜色等发生改变时，浏览器会对该元素进行重绘。  
为了减少重排和重绘，可以从以下几个方面着手。


## 批量增删节点
### 通过`elem.innerHTML`批量删除和添加子节点。  
### 通过`elem.insertAdjacentHTML`批量添加子节点。  
### 通过创建一个fragenment来批量添加子节点。  
```javascript
var fragenment = document.createDocumentFragenment();
for(var i = 0; i < 10; i++){
  var div = document.createElement('div');
  fragenment.addChild(div);
}
elem.appendChild(fragenment);
```
### 通过创建一个range来批量删除一组连续节点。  
```javascript
function removeChildren(parent, start, end){
  var range = document.createRange();
  var children = parent.children;
  range.setStartBefore(children[start]);
  range.setEndAfter(children[end]);
  children = null;
  range.deleteContents();
}
```  


## 批量修改节点和样式
### 通过修改class属性代替多次修改style属性  
不好的做法：
```javascript
elem.style.width = '200px';
elem.style.height = '200px';
```
较好的做法：
```css
.another-style{
  width: 200px;
  height: 200px;
}
```
```javascript
elem.className = 'another-style';
```

### 将节点隐藏或者删除后，批量修改，然后再显示或者添加到DOM树中。不管修改了多少次，只重新布局两次。
### 先克隆节点，然后对克隆出来的节点做相应修改，然后再将原节点替换下来。  
### 尽量缩小修改的影响
### 让对经常变化的节点脱离文档流，比如使用absolute或者fixed的position。这样，当该节点重排时不会影响其以后的兄弟节点。
### 少使用table布局，很小的改动会导致整个table的重新布局。  

## 访问DOM节点的某些属性
比如offsetTop, offsetLeft, offsetWidth, offsetHeight, scrollTop/Left/Width/Height, clientTop/Left/Width/Height, getComputedStyle()等等，参考[When does DOM access slow down rendering?](http://www.yiotabytes.com/2011/04/when-does-dom-access-slow-down.html)。文章中也提到浏览器越来越聪明，会尽量避免这些不太必要的重排重绘。我暂时还不知道怎么验证是否发生了重排重绘。  


留个问题，CSS3 Transition动画能不能减少重排呢？重绘肯定是少不了的。


