---
title: 年报系列分享之工具篇
---

如同前面的文章所讲，年报项目和其他项目有很大不同，开发过程中用到的工具也有所不同。这篇文章主要介绍我们在开发年报项目时用到的一些工具和方法。因为我们公司的设计师和前端都使用`mac`电脑，因此以下介绍的工具中有可能是`macOS`系统中特有的。

## 分解动画

不像其他我们常做的后端管理类网站，年报项目的动画非常多，设计师使用`Adobe Premiere`做好了动画导出为`MP4`格式的视频文件，我们再根据视频实现为网页动画，主要是CSS3动画，我们需要清楚地知道在动画从什么时候开始，持续多久。而视频文件播放时很快，很难察觉到这些信息。为此我们让设计师给我们做了动画分解，如下，

![](http://qiniu.wecode.club/blog/annual-report-tools-animation-break-down.png)

从动画分解中可以看出，动画为每秒30帧，并且设计师给出的时间也是按照“分:秒:帧”的格式给出的。为了方便获取毫秒数，我还特别在页面中注入了一个全局方法以方便获取毫秒数。需要注意的是计算持续时间不要直接使用“分:秒:帧”哦，还是老老实实换算成毫秒数再相减。因为这里的帧是30进制的，很容易出错，并且不容易被发现。
```ts
window.utils = {
    ms(frame: string) {
        const array = frame.split(':').map(s => Number(s));
        return array[0] * 60 * 1000 + array[1] * 1000 + Math.round((array[2] * 1000) / 30);
    }
}
```

在开发过程中还摸索到一个方法，就是`macOS`自带的`QuickTime Player`可以打开视频文件，并且可以通过左右两个方向键来实现按帧播放视频，对于我们研究动画效果非常方便。令我对`macOS`肃然起敬。

![](http://qiniu.wecode.club/blog/annual-report-tools-quick-time-player.gif)


## 实现动画

使用CSS3实现了动画以后，有时候会感觉怪怪的，但是因为动画速度很快，就难以发现问题。如果你想到的时候修改代码延长动画时长，那就Out了。修改代码后如果忘记恢复，可能造成一个Bug；还有页面中的多个动画可能互相配合，你修改了一个动画时长可能使得整个页面更加不协调，更加不容易发现问题了。有幸的是Chrome控制台有一个叫做`Animations`的面板，通过它可以暂停动画，也可以将动画速度调慢到4倍（25%）或10倍（10%），让你更加清楚地看到动画是如何执行的。通过它还可以每一个元素的动画，还可以即时调整动画，不要太方便，太强大。

![](http://qiniu.wecode.club/blog/annual-report-tools-chrome-animations.png)

还有一个`Rendering`面板也非常有用，我常用的两个功能是显示帧速`FPS meter`和显示高亮重绘区域`Paint flashing`。

打开`FPS meter`选项后，我们可以看到页面多了一个信息面板，上面显示帧速和GPU使用情况，通过帧速我们可以看到这个页面的动画是否流畅，动画是否需要优化等。

![](http://qiniu.wecode.club/blog/annual-report-tools-chrome-fps-meter.png)

打开`Paint flashing`选项后，如果某个元素被绿框遮盖，就说明它正在重绘。重绘是影响FPS的重要原因之一。当某个页面的FPS较低时，就可以去看看这个页面哪些元素正在重绘，能不能减少重绘？如果你有兴趣，可以参考谷歌的文档：[简化绘制的复杂度、减小绘制区域](https://developers.google.com/web/fundamentals/performance/rendering/simplify-paint-complexity-and-reduce-paint-areas?hl=zh-CN)。

![](http://qiniu.wecode.club/blog/annual-report-tools-chrome-paint-flashing.gif)

OK，就介绍这么多，以后想到了其他方法再来补充。