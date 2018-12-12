---
title: JavaScript 面试题
sage: true
---

1. ####什么是闭包？举个例子？
   闭包是指一个函数可以访问其生成时各级（非顶级）作用域中局部变量的能力。

```js
var lis = document.querySelector('#list li');
for (var i = 0; i < li.length; i++) {
    li.addEventListener(
        'click',
        (function(index) {
            return function() {
                console.log(lis[index].getAttribute('name'));
            };
        })(i),
        false
    );
}
```

在以上例子中，点击每个 li 时就会打印出该 li 的 name 属性。绑定在 click 事件上的匿名函数可以访问局部变量 index，就是利用了闭包原理。

1. ####请解释一下 JavaScript 的同源策略。
   同源是指相同的协议、域名、端口。
   最早用来阻止一个源下面的 js 去获取或者修改另一个源下面的文档属性，比如 iframe 加载另一个源下的页面。
   后来也被用于 AJAX 请求，是指一个源只能同一个源的服务器发起 AJAX 请求。

1. ####请解释 AJAX 的工作原理，以及其优缺点。
   传统的 Web 应用允许用户向服务器提交 Form 表单，服务器处理之后返回一个新的页面。这种做法既浪费带宽，因为前后两个页面很多相同之处，也增加了响应时间，增加了白屏时间，降低了用户体验。后来的 iframe 技术可以局部刷新页面，当时仍然没有根本改变以上状况，此时 AJAX 技术应运而生。
   AJAX 全称是 Asynchronous Javascript and XML，异步的 Javascript 和 XML 技术。AJAX 技术允许应用只向服务器请求必须的数据，并使用 Javascript 处理和渲染。节省带宽，提高了响应时间，基本不会白屏，提高了用户体验。
   其缺点也很明显，（1）破坏了浏览器的后退功能，最常用的解决办法是使用 URL 片段标识符（锚点，URL 中#后面的部分）来保持跟踪，允许用户回到指定的某个应用程序状态。或者使用现代浏览器都基本支持的 HTML5 规范中的 History API。

```javascript
// https://github.com/lichangwei/client-utils/blob/master/ajax.js
var xhr = new XMLHttpRequest();
xhr.open('get', '/url');
xhr.onreadystatechange = function() {
    if (xhr.state === 4) {
        console.log(xhr.responseText);
    }
};
xhr.send(null);
```

1. ####请解释 JSONP 的工作原理，以及它为什么不是真正的 AJAX。
   （1）客户端生成一个全局函数，比如 jsonp_callback。  
   （2）创建一个 script 标签访问某个资源，并加上参数?jsonp=jsonp_callback.  
   （3）服务器返回资源数据，格式为 jsonp_callback(资源数据)。  
   （4）浏览器解析服务器返回数据，即调用 jsonp_callback 函数，参数为资源数据。  
   JSONP 并不使用 XMLHttpRequest 对象加载资源，而是通过 script 标签把资源当做普通的 javascript 脚本来加载，所以不存在跨域问题，也不是真正的 AJAX。

1. ####你如何获取浏览器 URL 中查询字符串中的参数。

```javascript
// Using RegExp
function getQueryParam(key) {
    var search = window.location.search;
    if (!key || !search) return;
    key = key.replace(/\./g, '\\.');
    var match = search.match(new RegExp('[?&]' + key + '=([^&]*)'));
    if (match) {
        return decodeURIComponent(match[1]);
    }
}
```

```javascript
// Using String.split
function getQueryParam(key) {
    var search = window.location.search;
    if (!key || !search) return;
    var array = search.substr(1).split(/[\?&=]/);
    var index = array.indexOf(key);
    if (index !== -1 && index % 2 === 0) {
        return decodeURIComponent(array[index + 1]);
    }
}
getQueryParam('order');
```

1. ####请解释一下事件代理。
   将事件绑定目标元素的父元素上。在父元素的相应事件中判断 event.target 或其各级父元素是不是目标元素（一般通过 css 选择器判断），如果是，则在执行相应函数。  
   事件代理的基础是事件冒泡机制。对于部分不支持冒泡的事件时无法使用事件代理的。但是捕获阶段应该也是可以做到事件代理的，参考[通过 body 的 error 事件捕获页面中所有图片的 error 事件](./2013-06-06-Events-in-Capture-Phase.md)  
   使用场景：在绑定事件时，相应元素还没有生成，或者这类元素经常被替换掉，再或者元素个数太多，如果每个都分别绑定，太麻烦并且浪费内存。

1. ####描述一种 JavaScript memoization (避免重复运算)的策略。

```javascript
// https://github.com/lichangwei/client-utils/blob/master/memorize.js
/*
  JavaScript Momorization
  @param {string} func name of function / method
  @param {object} [obj] method's object or scope correction object
  @param {function} serialize method for arguments to get the hash value.
 */
function memorize(func, thisObject, serialize) {
    var cache = {};
    thisObject = thisObject || null;
    serialize =
        serialize ||
        function() {
            return Array.prototype.join.call(arguments, '_');
        };
    return function() {
        var hash = serialize.apply(null, arguments);
        if (!cache[hash]) {
            cache[hash] = func.apply(thisObject, arguments);
        }
        return cache[hash];
    };
}
```

1. ####什么是"use strict"？使用它的好处和坏处分别是什么？
   在严格模式下，以下使用方法收到影响。  
   （1）去除 with 关键字。  
   （2）非顶级作用域中，不使用 var 声明变量就会报错，防止误升级为全局变量。  
   （3）对于只读属性的修改报错，而不是静默失败。  
   （4）函数形参重名或者对象定义属性重名报错。  
   （5）函数中 this 不再默认指向全局对象 window。  
   （6）更加安全的 eval. eval 中声明的变量和函数都不会影响当前作用域。  
   （7）arguments.callee 和 arguments.caller 都不在可以使用。  
   从上面列举的严格模式下的部分特性可以看出，严格模式减少出错的机会，但是也限制 JavaScrip 语言的灵活性。

1. ####解释内存泄露并说明什么时候会出现内存泄露。
   内存泄露是指一块内存不会被释放，也不会被使用，直到浏览器进程结束。浏览器中采用垃圾自动回收机制，但是由于其实现的问题，可能会造成内存泄露。造成内存泄露大多是对象循环引用：

```javascript
function bindClickEvent() {
    var elem = document.getElementById('id');
    elem.onclick = function() {
        // do any thing
    };
}
```

以上函数的每次执行都会造成匿名函数和 elem 的循环引用。elem 显式引用匿名函数，匿名函数通过作用域链隐式引用 elem 元素。可以通过两种方法解决该问题，一是将 elem 设置为 null，二是将匿名函数置于 bindClickEvent 函数外面。两者都通过取消匿名函数对 elem 的隐式引用来解决该问题。
