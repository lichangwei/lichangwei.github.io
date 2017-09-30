---
title: 网页性能优化
---

提到网页性能优化，一般分为两部分，加载性能优化和交互性能优化。  

## 1.加载性能优化
当在浏览器中输入一个URL或者在其他页面点击一个链接时，浏览器要做以下工作：DNS查询，建立TCP链接，发送请求，接受响应，渲染页面，执行Load监听事件，至此页面加载就完成了。下面我们按照这个步骤思考如何优化页面加载性能？  

### 1.DNS查询
#### 1.减少DNS查询
#### 2.使用DNS Prefetching（DNS预获取）特性缩短DNS查询时间。
DNS查询一般占用带宽少，但是延迟可能很高，典型的一次DNS查询需要20-120ms。而DNS预获取为了获得较低的延迟，牺牲DNS查询次数。
参考：[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Controlling_DNS_prefetching)
参考：[DNS Prefetching Implications](http://www.pinkbike.com/news/DNS-Prefetching-implications.html)


减少请求
使用Data URI，
使用CSS Sprites
http://www.csdn.net/article/2013-09-13/2816925-CSS-Sprites-vs.-Data-URIs:-Which-is-Faster-on-Mobile?







## 2.TCP连接
### 1.使用keep-alive保持持久连接

### 2.交互性能优化 