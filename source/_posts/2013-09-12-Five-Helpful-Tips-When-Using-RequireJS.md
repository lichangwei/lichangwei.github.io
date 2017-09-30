---
title: 【翻译】RequireJS的五个有用的小技巧
---

[查看原文](http://tech.pro/blog/1561/five-helpful-tips-when-using-requirejs)

RequireJS--Javascript的文件和模块加载器--是Web应用中组织，管理，构建和加载Javascript的一种很强大的方式。我已经使用它好几年了，and while it's admittedly difficult to limit myself to only five, 我要是早点知道这几条提示就好了。注意，该文章假定你了解RequireJS，或者至少知道AMD，CommonJS和脚本加载器。RequireJS官方网站是一个让你知其然并且知其所以然的很好的站点。  

在这篇文章中，我们仅仅关注在RequireJS本身，而不包括r.js（RequireJS的优化工具）。我打算以后写一篇关于r.js的文章。  

## 1. 知道定义模块的几种方式
大多数使用RequireJS的人都知道以下这种定义模块的方法。  
```js
define(['dependencyA', 'dependencyB'], function(depA, depB){
  /* module code here using depA and depB */
  return myModule;
});
```
在98%的情况下，我使用上面的方法。但是还有其他几种选择，选择哪种，取决于你的需要。一个模块不需要任何依赖，那么你可以这么做：  
```js
define({
  someProp: 'Oooh, how interesting!',
  someMethod: function(){
    // do interesting work
    return compellingValue;
  }
});
```
如果这个模块没有任何依赖，但是你需要做一些准备工作或者保存私有状态，那么你可以这么做：  
```js
define(function(){
  var secretValue = 'seekret';
  // do other initialization work here...
  return {
    someProp: 'Oooh, how interesting!',
    tellMeSecrets: function(){
      // do interesting work
      return secretValue;
    }
  };
}
});
```

## 2. 怎么配合非AMD库一起工作
世界上有非常多的很好的Javascript库，大多数都没有遵循AMD规范，但是不用担心，你照样可以使用它们。从RequireJS 2.1.0版本开始，RequireJS有了一个叫做‘shim’的特性，让你设置你要用到的哪些不满足AMD规范的库，并且像其他AMD库一样加载进来。下面我们来看个非AMD库的例子。  
**加载没有依赖的非AMD库**  
你获取还记得Backbone和Underscore取消AMD支持的情景，多亏了shim特性，我们依然可以容易地在RequireJS中使用这些库。
```js
require.config({
  paths: {
    underscore: 'libs/underscore.min'
  },
  shim: {
    underscore: {
      exports: '_'
    }
  }
});
```
上面的require.config调用中我们提供了压缩后的underscore.js文件路径，后面在shim对象中，添加了一个和同样的属性，并且值是一个对象，它有一个exports属性，用来告诉RequireJS全局对象（如果是浏览器的话当然就window对象了）上某个属性就是这个模块实际值。既然Underscore把自己声明为window上的_属性，那么我们就让exports的值为_。从此以后，如果有一个AMD模块依赖Underscore，RequireJS会用全局对象的_值代替。  
这个例子很简单，因为Underscore没有任意依赖。那么如果加载一个有其他依赖的非AMD库吗？  
**加载有依赖的非AMD库**  
下面我们看看怎么加载依赖于Underscore和jQuery的Backbone吧。  
```js
require.config({
  paths: {
    jquery: 'libs/jquery.min',
    backbond: 'libs/backbone.min',
    underscore: 'libs/underscore.min'
  },
  shim: {
    underscore: {
      exports: '_'
    },
    backbone: {
      deps: ['jquery', 'underscore'],
      exports: 'Backbone'
    }
  }
})
```
Backbone的shim配置除了exports属性外还有deps属性，deps是一个包含了Backbone所有依赖的库名字数组，他们必须先于Underscore加载，一旦这些依赖加载完成之后，Backbone也会被加载，然后RequireJS就会从全局对象上获取Backbone属性来作为backbone模块的值。  
那么CommonJS又是怎样的呢？  
假定你要RequireJS中使用一个CommonJS模块，没问题，你可以定义一个模块，并提供一个带有三个参数的工厂函数，这三个函数分别是：require，exports和module。多数情况下你都可以忽略其他两个参数，而只考虑如何使用require参数。通过require参数可以通过CommonJS的语法获取一个模块，比如：
```js
define(function(require, exports, module){
  var cjs = require('myCommonJSModule');
  return {
    findDroids: function( droids ){
      var res = [], i = 0, len = droids.length;
      for( ; i < len; i++){
        if(cjs.isDroidWeareLookingFor(droids[i])){
          res.push(droids[i]);
        }
      }
      return res;
    }
  }
});
```
特别注意我接下来的一片关于r.js的文章，当你使用r.js优化工具时，也可以使用更多的CommonJS工具库。你可以选择将CommonJS模块转换成用deine()方法包裹起来的模块。你还需要知道，如果你的CommonJS模块有分支逻辑去有条件地调用require，那么转换方法就会失效。比如上面的myCommonJSModule有以下内部逻辑（参考代码片段中去思考是什么让它不能和RequireJS不能一起工作）：  
```js
// inside myCommonJSModule
module.exports = {
  isDroidWeAreLookingFor: function(droid){
    var finder;
    // OHSNAP! The conditinal logic around require
    // means we cannot wrap it with define in RequireJS
    if(isObiWanPresent){
      finder = require('forceFilter');
    }else{
      finder = require('normalFilter');
    }
    return finder.find(droid);
  }
}
```

## 3. CND Fallbacks
虽然CDN（内容分发网络）可以提高站点的加载性能，但是不希望万一CDN挂掉时，你的站点也会挂掉。幸好RequireJS可以容易地设置后备地址。一般配置如下：
```js
require.config({
  paths: {
    kendoui: 'http://cdn.kendostatic.com/2013.2.716/js/kendo.all.min.js'
  }
});
```
当然我们希望CDN永远不会挂掉，但是如果它挂掉了，我们可以通过修改配置来使用后备地址：
```js
require.config({
  paths: {
    kendoui: [
      'http://cdn.kendostatic.com/2013.2.716/js/kendo.all.min.js',
      'libs/kendoui/kendoui.min'
    ]
  }
});
```
我们给kendoui设置了一个地址数组，而不再是一个字符串，根据以上配置，RequireJS将会尝试从CDN上加载，如果失败了，再尝试从第二个地址加载。使用后备地址的确会使得脚本加载时间变长，但是总比站点不可用要好得多。  

## 4. 插件
 RequireJS中最好的一个增值点或许就是加载插件。加载插件可以用来加载各种非Javascript资源，就像其他依赖模块一样加载他们。最常见的是文本插件，它允许你加载一个普通文本文件（比如HTML或者CSS等），这对加载模板特别有用。为了让你理解它多么有用，让我们看看今天人们是怎么使用它的。  

 下面的代码片段展示了Backbone视图怎样获取unders.js模板内容，预编译，然后渲染页面，但是代码中你却看不到RequireJS/text插件的痕迹。  
```js
var MyView = Backbone.View.extends({
  initialize: function(){
    this.template = _.template( $('generated-gif-template').text() );
  },
  render: function(){
    this.$el.html( this.template( this.model.toJSON() ));
  }
})
```
注意到这个视图应该放在页面里面。很多人会添加一个`script`标签，并给出一个假冒的`type`属性（这样就不会被当做Javascript执行），将模板内容放进去，如下：  
```html
<script type="text/underscore-template" id="generated-gif-template">
  <span class="glyphicon glyphicon-remove"></span>
  <a href="<%= dataURL %>" download="<%= id %>">
    <span class="glyphicon glyphicon-save"></span>
  </a>
  <img src="<%= dataURL %>" class="img-thumbnail gif-item">
  <div>
    <input style="display:none" type="text" id="fileName" value="<%= fileName %>" />
    <span id="lblFileName"><%= fileName %></span>
  </div>
</script>
```
这方法肯定是可行的，但是有一些缺点。把模板内容嵌套在script标签中会丧失语法高亮和其他IDE提供的功能。这个模板必须出现在所有要使用它的页面。在一个多页面的站点中，这意味着你可能要多次拷贝模板内容，或者额外的构建步骤将模板嵌入每个目标HTML页面。在AMD应用中，像这样将模板保存在DOM中意味着该模块除了显示传递给它的依赖之外，还依赖其他东西。这使得你的应用比较脆弱。理想情况下，我们希望将模板传递给我们的模块，就像其他的脚本依赖一样。这就是文本插件产生的原因。  
```js
define([
  'backbone',
  'text!templates/generated-gif.html'
], function(Backbone, tempalte){
  return Backbone.View.entend({
    initialize: function(){
      this.template = _.template(template);
    },
    render: function(){
      this.$el.html( this.template( this.model.toJSON()));
    }
  });
});
```
我们将模板路径作为依赖加进来，用`text!`作为前缀，来告诉RequireJS使用一个叫做text的插件来处理。模板的文本内容作为`template`参数传递给模块。使用这个方法有如下好处：  
（一）开发过程中，模板作为一个单独的文件存在，因此我们使用IDE带给我们的便利（语法高亮等），还避免了我们必须穿过数百行甚至更多的HTML代码去编辑代码。  
（二）模板可以和其他模块同等对待，显示传递到模块中，让模块对模板的依赖更加显而易见。这和当模块执行时期望模板在DOM中正好相反。  
当我们使用r.js构建和优化RequireJS应用时，模板可以和其他模板一起部署，因为文本插件可以有效地将模板包装在模块定义调用中，创建一个返回模板文件内容的模块。  
以上就是对文本插件的惊鸿一瞥，下面是其他一些插件。  
[i18n](http://requirejs.org/docs/api.html#i18n) - 国际化  
[image](https://github.com/millermedeiros/requirejs-plugins) - 像加载其他模块一样加载图片  
[mdown](https://github.com/millermedeiros/requirejs-plugins/) - 加载markdown文件，它会给你编译成html  
[font](https://github.com/millermedeiros/requirejs-plugins) - 加载Web字体  

## 5. 故障检修小技巧  
你可以在代码中使用以下API，我还发现在Chrome控制台中也非常有用。  
`require.defined(moduleId)` - 返回true如果该模块已经被定义并且可用。
`require.specified(moduleId)` - 返回true如果该模块被其他模块依赖。该函数返回true并不意味着该模块可用。
`requirejs.s.contexts._.config` - 我从[Vernon Kesner](https://twitter.com/vernonk)那里学来的这招。这是一个技术上的后门，文档中没有说明的方法，所以它可能在没有任何警告的情况下被修改或者去除。但是它返回一个非常有用的包含配置信息的对象。  
以下是在Chrome控制台中它返回结果：  
![Chrome控制台中它返回结果](http://tpstatic.com/img/usermedia/_TBvB2AB50KM-TASuLKFsw/w645.png)  
以上就是我在Devlink用来演示[Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Worker)的一个[生成GIF示例应用](https://github.com/ifandelse/gif-stitch)中调用requirejs.s.contexts._.config返回的结果。你可以看到所有相关配置数据：根URL，路径，shim配置等。
当进行Debug时其他两个关键是`errbacks`和`requirejs.onError`方法。  
`RequireJS ‘errbacks’`
当你调用`require`时，可以传入第三个参数-发生错误时的回调，允许你对错误做出反应，而不是产生一个未捕获的异常。比如：  
```js
require(
  [ 'backbone' ], 
  function ( Backbone ) {
    return Backbone.View.extend({ /* your magic here */ });
  }, 
  function (err) {
    /* 
      err has err.requireType (timeout, nodefine, scripterror)
      and err.requireModules (an array of module Ids/paths)
      Inside here you could requirejs.undef('backbone') to clear
      the module from require locally - and you could even redefine
      it here or fetch it from a different location (though the
      fallback approach earlier takes care of this use-case more succinctly)
    */
  }
);
```
`requirejs.onError`  
RequireJS有一个全局的错误处理函数，它可以捕捉到所有未被`errbacks`处理的异常。代码如下：  
```js
requirejs.onError = function (err) {
  /* 
    err has the same info as the errback callback:
    err.requireType & err.requireModules
  */
  console.log(err.requireType);
  // Be sure to rethrow if you don't want to
  // blindly swallow exceptions here!!!
};
```

## 总结
无论你是刚接触RequireJS或者已有很多经验的用户，就像我前面提到的，我将会在另一篇文章中介绍`r.js`，敬请期待。你是否发现了其他有用的关于RequireJS使用的技巧，方法和API，期待你的交流。  
