
1. ####解释什么是Deferred，以及优点是什么。
Deferred，意为延迟执行，是jQuery的回调函数解决方案，对所有的异步操作提供统一和更加方便的编程接口。使用Deferred实现的$.ajax有以下几种有点。（1）链式调用（2）指定多个回调（3）为多个操作指定回调  
```javascript
$.ajax('test.html')
  .done(function(){console.log('Done.')})
  .fail(function(){console.log('Fail.')})
  .done(function(){console.log('OK.'));
$.when($.ajax('test1.html'), $.ajax('test2.html'))
  .done(function(){console.log('Done.')})
  .fail(function(){console.log('Fail.')});
```

1. ####你如何给一个事件处理函数命名空间，为什么要这样做？
```javascript
// 主要用于jQuery插件，当需要取消某插件的效果时，可以通过unbind('.namespace')一次性删除该插件绑定的所有事件，而不会影响到其他插件或者用户手动绑定的事件。
var $btn = $('XXX');
// 比如我们写了一个实现拖拽的插件，该插件监听了mousedown，mousemove和mouseup事件。
$btn.bind('mousedown.dragdrop', function(){});
$btn.bind('mousemove.dragdrop', function(){});
$btn.bind('mouseup.dragdrop', function(){});
// 当某个节点不支持拖拽时，取消拖拽相关的事件监听，就可以使用如下方法，一次性删除。
$btn.unbind('.dragdrop');
```

1. ####请指出'$'和'$.fn'的区别？或者说出'$.fn'的用途。

1. ####解释chaining。
```javascript
// 链式调用，如果对象的方法不需要一个明确意义的返回值，那么可以返回自身。
$('XXX').css('width', '100px').show().click(function(){});
```

1. ####你知道那些针对jQuery的优化方法。
（1）将一个jQuery对象保存起来，如果多次使用的话。

1. jQuery中prop和attr方法的异同？

