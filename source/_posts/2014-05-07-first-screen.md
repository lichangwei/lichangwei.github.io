---
title: 如何计算首屏加载时间？
---

在网站开发中，首屏是我们最需要关注的指标之一，它基本代表着用户眼中的网站加载时间。但是如何衡量它呢？

在我们公司有这样的一种做法，将以下代码放入需要首屏最后一个元素后面

```html
<script type="text/javascript">
    /* <![CDATA[ */
    PAGE_TIMING.firstScreenImage = new Image();
    PAGE_TIMING.firstScreenImage.onload = function() {
        PAGE_TIMING.firstScreen = new Date().getTime();
    };
    PAGE_TIMING.firstScreenImage.src = 'http://sample.com/wimg/monitor/first-screen.png';
    /* ]]> */
</script>
```

其中`first-screen.png`是用了缓存。我们将`PAGE_TIMING.firstScreen`作为首屏加载时间，这样也只是一个近似时间。不知道各位看官你们是如何处理的呢？
