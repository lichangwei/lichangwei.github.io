---
title: 蓝光机WebApp-无尽列表优化
---

## 背景介绍
该项目是一个运行于蓝光机内置浏览器Opera之上的单页面WebApp，输入关键字，跨服务（视频，音频，Youtube等）查询媒体内容。由于所有操作都是通过遥控器完成，所以通常可以使用的操作基本只有按键：上下左右以及返回键，确定键。为了显示不确定数量的媒体项，采用了通过向下/向上键来控制显示更后面/更前面的记录。  
使用的框架包括，使用[requirejs](http://requirejs.org/)动态加载脚本，使用[jQuery](http://jquery.com/)操作DOM，使用[doT](http://olado.github.io/doT/)作为模板引擎。  

## 优化历程
分析一：根据一般的WebApp开发经验知道，不可能在一个页面中显示所有的媒体项。有必要分页显示。但是每页显示多少呢？通过对蓝光机的实机测试，每页显示30个项目，另外根据UI设计每屏最多显示12条。  

**方案一**：首先显示第一个30条（0-29），随着用户按下向下键，当要显示第30条记录的时候，使用`$container.html($(str))`显示第18-47条，当需要显示第48条的时候，使用`$container.html($(str))`显示第36-65条……依次类推下去  
现象一：桌面浏览器没有问题，但在蓝光机中会出现闪烁，即在某个时刻页面中没有任何内容。所有的蓝光机概率为**~95%**  
  
分析二：这种现象的原因很简单，`$container.html($(str))`确实做了一个清空容器的操作，如果性能较差，会出现闪烁现象。  
  
**方案二**：显示新纪录时，当前显示在页面中12条记录保持不变，使用`$container.find(':lt(18)').remove()`去掉前面18条记录，再使用`$container.append($(str))`添加后面的18条记录。  

现象二：桌面浏览器没有问题，但在性能较好的蓝光机中出现闪烁的概率为 **~20%**，在较差的蓝光机中概率为 **~50%**  
  
分析三：以上仍然使用jQuery封装的find，remove以及append方法，这些方法的实现还是相当复杂，做了太多无用的工作。  
**方案三**：使用原生的DOM操作方法`document.createDocumentFragment()`创建一个文档片段，批量添加记录。  
现象三：桌面浏览器没有问题，但在性能较好的蓝光机中出现闪烁的概率为 **~1%**，在较差的蓝光机中概率为 **~15%**  
  
分析四：仍然使用了`$container.find(':lt(18)').remove()`去掉前面18条记录，这里`remove`方法可以导致$container这部分内容18次的重排重绘。 这是因为并没有找到和`document.createDocumentFragment`相似的可以批量删除DOM节点的方法。 在桌面浏览器，移动浏览器常用的方法，比如首先将$container从DOM树中删除，执行一系列的删除操作之后再添加到DOM树中，都会导致更频繁的闪烁现象。  
在即将放弃该项优化的时候，脑子里突然冒出了一个从来没有使用过的方法。既不清楚能够解决该问题，也不知道如何使用该方法，只知道这个方法通常用在富文本编辑器中，好像可以批量操作元素。  

**方案四**：使用`document.createRange`批量删除元素。  

``` javascript
function removeChildren(parent, start, end){
  var range = document.createRange();
  var children = parent.children;
  range.setStartBefore(children[start]);
  range.setEndAfter(children[end]);
  children = null;
  range.deleteContents();
}
```

现象四：桌面浏览器以及所有的蓝光机中均没有闪烁现象。  

最后又发现`elem.insertAdjacentHTML`的执行效率在蓝光机上优于`document.createDocumentFragment`，基本上前者耗时是后者的 **50%**，但是不解的是在Chrome26/Windows7上前者耗时略高于后者。由于目标设备是蓝光机，所以这次优化成功，可以使用`elem.insertAdjacentHTML`。  
`elem.insertAdjacentHTML`第一个参数是字符串，表示插入位置，共有四个可选值，如下表说明（来源：Javascript权威指南，P379）。第二参数也是字符串，表示要插入的HTML字符串。

```html
    <div id="target">This is the element content</div>
   ↑                ↑                           ↑     ↑
beforebegin     afterbegin              beforeend   afterend
```
