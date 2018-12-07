---
title: 网页性能优化
---

提到网页性能优化，一般分为两部分，加载性能优化和交互性能优化。

## 1.加载性能优化

当在浏览器中输入一个 URL 或者在其他页面点击一个链接时，浏览器要做以下工作：DNS 查询，建立 TCP 链接，发送请求，接受响应，渲染页面，执行 Load 监听事件，至此页面加载就完成了。下面我们按照这个步骤思考如何优化页面加载性能？

### 1.DNS 查询

#### 1.减少 DNS 查询

#### 2.使用 DNS Prefetching（DNS 预获取）特性缩短 DNS 查询时间。

DNS 查询一般占用带宽少，但是延迟可能很高，典型的一次 DNS 查询需要 20-120ms。而 DNS 预获取为了获得较低的延迟，牺牲 DNS 查询次数。
参考：[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Controlling_DNS_prefetching)
参考：[DNS Prefetching Implications](http://www.pinkbike.com/news/DNS-Prefetching-implications.html)

减少请求
使用 Data URI，
使用 CSS Sprites
http://www.csdn.net/article/2013-09-13/2816925-CSS-Sprites-vs.-Data-URIs:-Which-is-Faster-on-Mobile?

## 2.TCP 连接

### 1.使用 keep-alive 保持持久连接

### 2.交互性能优化
