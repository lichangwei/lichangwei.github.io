---
title: HTML5应用缓存与百度地图服务
---

本文假设你基本了解 HTML5 应用缓存。

前几天，一个同事联系到我，说他们项目想使用 HTML5 的应用缓存，但是遇到了脚本执行错误问题，但是如果去掉 manifest 配置，即不使用应用缓存，则一切正常运行。我让他将项目代码简化一下，剥离业务相关部分，然后打包发给我。

在浏览器中打开该页面以后，发现如下两个错误：

```
Uncaught ReferenceError: BMap is not defined index.html:18
GET http://api.map.baidu.com/getscript?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2&services=&t=20130916115231
```

以上两个错误很明显是脚本加载失败造成的。

通过代码可以看到，项目中使用了百度地图，使用方法是引入一个`script`标签.

```html
<script src="http://api.map.baidu.com/api?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2"></script>
```

以上`script`标签从服务器端获取的代码是：

```js
(function() {
    window.BMap_loadScriptTime = new Date().getTime();
    document.write(
        '<script type="text/javascript" src="http://api.map.baidu.com/getscript?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2&services=&t=20130916115231"></script>'
    );
})();
```

我们可以看到它使用了`document.write`方法，插入另一个`script`标签。然而在`manifest`文件中，却没有说明该脚本应该如何处理。

```
CACHE MANIFEST
NETWORK:
http://api.map.baidu.com/api?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2
```

这时候浏览器尝试从应用缓存中寻找`http://api.map.baidu.com/getscript?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2&services=&t=20130916115231`却没有找到，所以脚本加载失败。因为这个脚本网址不是公开 API 的一部分，今天是这个 URL，明天也许就是另一个 URL 了，所以我们不能写在`manifest`文件里面。再者地图相关资源肯定不适合应用缓存，一是我们不能枚举需要调用的脚本，图片和其他资源，从而罗列在`CACHE:`配置中，二是应用缓存在每个域下只有 25M 的配额，甚至更少。那么对于这种不固定的网址或者不可枚举的网址该怎么处理呢？

这时候我们有两种做法，第一种比较简单，在`manifest`文件中 NETWORK 一栏中使用`*`这个通配符，表示如果某个 URL 没有显式定义出来，则从互联网上获取该资源。于是`manifest`文件就变成：

```
NETWORK:
*
```

另外一种做法就是将这些不固定的网址（在这里，是百度地图服务）封装在一个固定地址的页面中，不使用应用缓存，并且通过 iframe 元素引入。  
比如：

```
NETWORK:
map.html
```

其中 map.html 负责渲染地图，并且不使用应用缓存。

现在问题看似解决了，但是我们的目的是使用应用缓存，那么当百度地图服务不可用（没有网络或者百度地图服务宕掉）时，打开页面是怎样的呢？我关闭了网络连接后，刷新页面，看到是一个块空白，以及如下错误：

```
Application Cache Error event: Manifest fetch failed (-1) http://192.168.0.100/github/map/map.manifest map.html:1
Uncaught ReferenceError: BMap is not defined map.html:18
GET http://api.map.baidu.com/api?v=1.5&ak=24fddd5bf8d6cbe100c40dfa9aed10d2
```

直接给用户一个空白的地图似乎不是那么友好，更好的一种做法就是在使用一张静态的百度地图的截图代替，或者通过文字告诉用户，地图加载失败。

这时候有两种做法，一种做法是判断 BMap 是否定义，如果已定义则认为百度地图服务可用，否则认为不可用，此时可以提示用户地图服务不可用。这种做法很简单，这就不多说了。另一种做法更加适合上面使用 iframe 的情形，使用用应用缓存中的 FALLBACK 配置项。在 FALLBACK 配置中，每一行包括两个 URL，用空格分隔。第一个 URL 是正常情况下的资源地址，第二个 URL 是后备资源地址，当前者不可用时，由后者顶上。

使用 FALLBACK 时，我们可以定义一个 nomap.html 作为百度地图 API 的后备处理方案。代码如下：

```
FALLBACK:
map.html nomap.html
```

> **FALLBACK 有个坑，就是 FALLBACK 中配置中第一个 URL 不能出现在 NETWORK 中，否则 FALLBACK 配置可能失效。**

## 总结：

1. 可以使用`NETWORK`中统配符`*`来告诉浏览器，对于那些未显式指定的资源总是从网络上获取。
2. 对于`iframe`让应用的一部分使用应用缓存，另一部分不使用。
3. 配合`FALLBACK`，可以让网络不可用时或者服务不可用时，界面更加友好。
