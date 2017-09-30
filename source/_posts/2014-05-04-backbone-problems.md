---
title: Backbone中的几个小问题
---

Backbone用起来还行，但是在某些方面还是有不方便之处。比如，  
1. 需要手动将数据填充到DOM中，需要手动从DOM中抽取数据  
```js
// 手动填充数据
this.$el.html(this.template(this.model.toJSON()));
// 手动收集用户输入的表单数据
var arr = $('form-id').serializeArray();
```

2. 需要重复绑定事件，由于其模板引擎是基于字符串的，所以在model更新时，需要替换掉相应DOM树，而不是在原有DOM上更新，每次DOM替换后，都需要重复绑定事件。当然你可以选择使用事件代理，但是并不是什么时候都可以使用事件代理的，比如你依赖的组件不支持事件代理，这时候就很郁闷啦。  

3. 同样由于其模板引擎是基于字符串的，所以如果你监听了model的change事件，那么你需要注意，你需要尽可能的批量更新model，否则会导致连续的多次大块DOM替换。
```js
// 不好，可能导致多次大块DOM渲染
model.set({'name': 'Bu Feng'});
model.set({'age': 28});

// 好，只有一次DOM渲染。
model.set({
  'name': 'Bu Feng',
  'age': 28
});
```
除了这一点外，还需要考虑缩小监听范围，比如
```js
// 仅仅监听isShown字段是否被修改，忽略其他字段
model.bind('change:isShown', this.display);
```

今天又遇到一个问题，先说一下背景吧。最近网站由于故障频发，要求后端必须灰度发布，就是首先发布到线上的一台机器，测试，如果没有问题再发布到所有机器上，否则回滚；而且前端和后端是分别发布，并且前端没有灰度发布。这样一来，前端需要同时兼容后端的新旧两个版本。所以前端发布时需要测试两种情况：新后端+新前端，旧后端+新前端。在新后端+新前端测试通过后，测试旧后端+新前端时，发现了`ReferenceError: XXX is not defined`这样的错误。根据Chrome控制台提供的信息可以看到，是由于**Backbone的模板引擎是通过with实现的**。我们知道with有一个问题：在with作用域内引用对象上不存在的属性，则会报出ReferenceError错误。比如：
```js
var obj = {a: 1};
with(obj){
  console.log(a); // OK
  console.log(unexistedAttr); // ReferenceError: unexistedAttr is not defined
}
```
在新版本中某个Ajax请求中增加一个字段，假定为f，在前端模板中通过`<% if(f){ %> ... <% } %>`引用了该字段，没有任何问题，而在旧版本后端中没有该字段，就会报`ReferenceError: f is not defined`错误。

针对这个问题，一般可以给model设置默认属性值，但是总归还是比较麻烦的。也幸好Chrome的Debug能力很强，通过点击错误就看到了编译后的模板方法。如果使用Firefox浏览器，只能看到报了`ReferenceError: f is not defined`，恐怕你还很难猜测到是模板里引用了该变量导致的呢。

顺便说一下，我不太喜欢backbone模板引擎的语法的，使用`<% %>`看起来和html标签太像，看起来总是觉得不太舒服。还是比较喜欢doT的语法风格，
```html
<div>Hi {{=it.name}}!</div>
<div>{{=it.age || ''}}</div>
```
在编辑器中使用了语法高亮之后，看起来好整洁。

以上就是我使用Backbone遇到的一些问题，以后有新问题再补充。