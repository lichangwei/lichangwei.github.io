---
title: 【Chrome Extension】如何获取Extension的版本号
---

Chrome中有两种扩展应用类型，一种是Extension，它通过往页面中添加script脚本来达到其目的。另外一种是Packaged Application，它是基于Chrome的本地应用。具体用法以后再总结。  

这两种扩展都是需要用户手动安装的，也是可以自动更新的。但是有些时候，我们还是需要去检查Extension的版本。下面我们就看看如何去做？  

首先，我们知道，在编写Extension时，我们都需要写一个`manifest.json`文件，它里面包含着关于这个Extension的信息。比如`name`，`description`，`version`等很多信息。那么我就可以通过获取该文件来获取本扩展的版本。  

那么如何获取`manifest.json`文件呢？  

首先我们知道Extension中可以通过`XMLHttpRequest`来获取资源文件，而`manifest.json`也可以看做是普通的一个资源文件，那么我们就可以通过`XMLHttpRequest`获取`manifest.json`。  
```javascript
(function(){
  var url = 'manifest.json';
  var xhr = new XMLHttpRequest();
  xhr.onload = function(){
    var manifest = JSON.parse(xhr.response);
    var version = manifest.version;
    // do something here
  };
  xhr.open('GET', url, true);
  xhr.send();
})();
```
以上方法放在Extension的content script中，报如下错误：`GET http://[ip]:[port]/manifest.json 404 (Not Found)`。这个很容易理解，因为在Extension中，该脚本是通过script标签插入到DOM中执行的。那么我们如何来获取manifest.json的实际路径呢？答案是`chrome.extension.getURL`，这个API就是根据扩展中资源文件的相对路径来获取其绝对路径。而在Packaged App中则可以直接使用相对路径。所以在Extension中的做法是：  
```javascript
(function(){
  var url = chrome.extension.getURL('manifest.json');
  var xhr = new XMLHttpRequest();
  xhr.onload = function(){
    var manifest = JSON.parse(xhr.response);
    var version = manifest.version;
    // do something here
  };
  xhr.open('GET', url, true);
  xhr.send();
})();
```
还要记住，我们需要在`manifest.json`中声明`manifest.json`是一种可以访问的资源。  
```json
{
  "web_accessible_resources": ["manifest.json"]
}
```
否则我们会看到如下错误：
```
Denying load of chrome-extension://ongmmjdilaoifhglgibpinckpckeclch/manifest.json. Resources must be listed in the web_accessible_resources manifest key in order to be loaded by pages outside the extension. 
```

以上我们把`manifest.json`当做一个普通的资源文件来访问。那么`manifest.json`作为一种很关键很通用（所有Extension中都有该文件）的一个配置文件，那么有没有更简单的方法呢？答案是肯定的。在最新的Chrome（v22+）中提供了`chrome.runtime.*`API，他们提供了获取background页面，manifest.json文件，以及和其他Extension通信的各种API。代码如下：  
```javascript
var manifest = chrome.runtime.getManifest();
var version = manifest.version;
// do something here
```

很明显，直接使用`chrome.runtime.getManifest`来获取Extension的版本号，最为简单，代码少，还是同步的。

总结：文章总结了获取Extension版本号（在`manifest.json`中）的两种方法，一种把`manifest.json`当做普通资源文件访问，一种直接使用`chrome.runtime.getManifest`访问，后者更为简单易用。