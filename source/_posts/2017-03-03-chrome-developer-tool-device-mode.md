---
title: Chrome 开发者工具 - 设备模式
---

使用 Chrome 开发者工具的设备模式可以大大减低开发移动优先响应式网站的难度，下面我们开始学习如何使用它模拟各种设备以及开发完全响应式的网站。

**概述**

 - 使用设备模式的屏幕模拟器测试网站的响应式特性
 - 自动保存设置，方便你以后继续使用。
 - 设备模式不能代替真实设备，请熟知它的局限性。

## 1 打开设备模式
![device-mode-initial-view.png-101.8kB][1]
通过切换“设备模式”按钮或者使用快捷键`Command+Shift+M (Mac)`或`Ctrl+Shift+M (Windows, Linux)`

## 2 `viewport`控制器
![device-mode.png-30.8kB][2]
`viewport`控制器共有两种模式：
**完全响应式**：自由地改变`viewport`尺寸。
**特定的设备**：模拟设备`viewport`尺寸和设备特性。

### 2.1 完全响应式
我们推荐你将完全响应式模式作为你的默认模式，在开发过程中，使用它频繁调整`viewport`的尺寸来实现一个完全的响应式设计来适配未知或未来的设备类型。
你可以拖拽调整大小的手柄或单击菜单栏中值进行细粒度的控制。
也可以直接选择`Mobile S - 320px`, `Mobile M - 375px`, `Mobile L - 425px`, `Tablet - 768px`, `Laptop - 1024px`, `Laptop L - 1440px`, `4K - 2560px`。

### 2.2 特定的设备
在开发即将结束时，通过使用特定设备模式，可以查看你的网站在特定设备上被渲染成什么模样，比如特定的`iPhone`或`Nexus`。

**内置设备集**
设备下拉框中已经内置了一组目前最流行的设备，选择了某个设备之后，Chrome开发者工具就会模拟该设备并模拟网站在该设备上会如何渲染。

 - 模拟用户代理（User Agent）
 - 模拟设备分辨率以及DPI（设备像素比）
 - 模拟触摸事件（如果设备支持的话）
 - 模拟滚动条和`viewport`
 - 对于没有定义`viewport`的页面自动调整文字大小

![select-device.png-19.6kB][3]

**添加自定义设备**
如果遇到极端情况，需要兼容特殊设备，你也可以添加自定义设备。点击设备列表下面的`Edit ...`选项即可打开以下窗口，你可以勾选更多设备和添加自定义设备。
![custom-device.png-90.6kB][4]


**设备状态和方向**
![change-orientation.png-6.8kB][5]
当模拟特定设备时，设备模式工具栏会出现一个额外的功能，用来切换设备方向横向（landscape）或纵向（portrait）。
对于某些机型，不仅能切换设备方向，还能模拟特定设备状态。比如对于`Nexus 5X`还可以模拟以下状态：

 - 默认的浏览器界面
 - 带有地址栏
 - 键盘处于打开状态

![change-device-state.png-13.5kB][6]

**缩放到合适尺寸**
有时候你需要测试一个比你浏览器窗口分辨率还大的设备，这时候“缩放”这个选项就非常有用了。适合浏览器窗口（Fit to Window）会自动设置缩放级别以最大程度低利用浏览器空间，而特定的缩放比例可以用来测试DPI对图像显示效果的影响等是很有帮助的。

**可选控制项**
通过单击设备工具栏右侧三个点可以启用和修改可选控制项，目前选项包括：

 - 是否显示设备边框
 - 是否显示媒体查询
 - 是否显示刻度尺
 - 设置设备像素比
 - 设置设备类型（移动/桌面，是否支持触摸）
 - 模拟网络状况
 - 截屏

![show-media-queries.png-139.6kB][7]

**显示设备外框**
只有当你选择了某些设备（比如 iPhone 6）之后才能启用该特性，启用以后你将能够看到页面被放在设备外壳里面。
![www.google.com-(iPhone 6).png-34.9kB][8]

**显示媒体查询**
[媒体查询](https://developers.google.com/web/fundamentals/design-and-ui/responsive/fundamentals/use-media-queries)是响应式设计的重要组成部分，通过点击`Show media queries`可以看到媒体查询查看器，该工具会检测你样式中的媒体查询，并在顶部标尺中将其显示成彩色条。
![media-query-inspector-ruler.png-6.9kB][9]

 - 蓝色代表最大宽度`max-width`
 - 绿色代表宽度区间`min-width ~ max-width`
 - 黄色代表最小宽度`min-width`

单击媒体查询条可以调整`viewport`大小来查看样式。
右击媒体查询条可以在源码中查看该媒体查询的定义。
![reveal-source-code.png-77.8kB][10]
 
**显示刻度尺**
![rule.png-48.2kB][11]

**显示设备像素比**
如果你想通过非视网膜屏设备模拟视网膜屏设备，或者反过来，那么这个功能就是为你准备的。设备像素比就是逻辑像素和物理像素的比例。视网膜屏设备比如`Nexus 6P`比一般设备的像素密度更高，这会影响视觉内容的清晰度和大小。
下面这些Web上的案例会受到设备像素比的影响：

 - CSS媒体查询：
    `@media (-webkit-min-device-pixel-ratio: 2) { ... }`
 - CSS[`image-set`](http://dev.w3.org/csswg/css-images/# image-set-notation)
 - `img`的[`srcset`](https://developers.google.com/web/fundamentals/design-and-ui/media/images/images-in-markup)属性
 - `window.devicePixelRatio`属性

在真正的视网膜屏上，低DPI（每英寸点数）资源看起来像素化，而高DPI的资源看起来很清晰，为了能够在标准的显示器上模拟这中效果，可以将DPR设置成2并缩放视窗，一个2X资源看起来仍然清晰，而1X的资源看起来就会像素化。

**设备类型**
单击`Add device type`，设备工具栏将会出现一个选项“Mobile”，共有以下几个可选值：`Mobile`，`Mobile(not touch)`，`Desktop`，`Desktop(touch)`，修改这项设置将会影响`viewport`，是否模拟触摸事件，用户代理字符串。因此如果你想为左面浏览器创建一个响应式网站并且向测试鼠标悬浮效果，请使用完全响应式模式并将设备类型切换到`Desktop`。
提示：你还可以在`Network conditions`修改用户代理。

**模拟网络状况**
单击`Add network throttling`，设备工具栏将会出现一个选项`No throttling`，通过这个选项可以模拟不同网络下页面加载效果。
![network.throttling.png-71.5kB][12]

## 3 模拟传感器：地理定位和重力感应

大多数台式机都没有GPS芯片和重力传感器，因而很难测试，但是 Chrome 开发者工具提供了传感器模拟器使得测试更加容易。

打开方法：
1. 打开开发者工具主菜单
2. 点击`Customize and control DevTools`- `More tools` - `Sensors`菜单项
![navigate-to-sensors.png-17.9kB][13]


### 3.1 模拟地理定位
移动设备一般使用GPS芯片进行定位，在传感器面板中，你可以使用[地理定位 API](http://www.w3.org/TR/geolocation-API/)模拟位置。你可以使用预置的几个城市或手动输入经度纬度，甚至还可以模拟定位失败的情况，这里有一个地理定位API的[例子](https://jsfiddle.net/lichangwei/mc0ep3u9/)，可以实验一下。
![geolocation.png-32.4kB][14]

### 3.2 模拟重力感应（加速度）
![orientation.png-16kB][15]
在浏览器中打开以下链接实验以下吧。[重力感应代码示例](https://www.ikdoeict.be/apps/leercentrum/courses/ws1-cws-course-materials/demos/06_html5_js/orientation.html)，切换方位看看页面如何变化。

## 4 参考资料
1. [Chrome开发者工具官方介绍](https://developers.google.com/web/tools/chrome-devtools/)，以上大部分内容来自该文档，部分甚至仅仅做了翻译，强烈推荐大家经常到该网站看看。


  [1]: http://static.zybuluo.com/lichangwei/uzwqzy5zn084re2j58aebu2c/device-mode-initial-view.png
  [2]: http://static.zybuluo.com/lichangwei/8u1ihmm4azuhwi4ssb9krruu/device-mode.png
  [3]: http://static.zybuluo.com/lichangwei/e3qyenwnwnyqg3z93k6kcylk/select-device.png
  [4]: http://static.zybuluo.com/lichangwei/1vho3slc17ql6366sjfafxpd/custom-device.png
  [5]: http://static.zybuluo.com/lichangwei/80mo0w30u799t46y9mgv8kvj/change-orientation.png
  [6]: http://static.zybuluo.com/lichangwei/xjk6aez20cl4mvf0fdijci9u/change-device-state.png
  [7]: http://static.zybuluo.com/lichangwei/krz2ci9gli2120g2g2afbrfi/show-media-queries.png
  [8]: http://static.zybuluo.com/lichangwei/akjj90o538yceskczug4lwkr/www.google.com-%28iPhone%206%29.png
  [9]: http://static.zybuluo.com/lichangwei/am40nh3xrcat33n5itno5y3h/media-query-inspector-ruler.png
  [10]: http://static.zybuluo.com/lichangwei/mdadktsfta7e8v5mdgaxcwwi/reveal-source-code.png
  [11]: http://static.zybuluo.com/lichangwei/q12f21enmo6laq9nqhi3mmju/rule.png
  [12]: http://static.zybuluo.com/lichangwei/g9qo71cb23v5g0omwp7hlm7r/network.throttling.png
  [13]: http://static.zybuluo.com/lichangwei/37dklb3tf2etn9t0miuyet6i/navigate-to-sensors.png
  [14]: http://static.zybuluo.com/lichangwei/4p8rz2bqkqfi2no661kz26et/geolocation.png
  [15]: http://static.zybuluo.com/lichangwei/qoclusfnvrctjib2ih1ilfo9/orientation.png