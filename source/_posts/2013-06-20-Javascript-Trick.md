---
title: Javascript的坑
---

以下题目摘自[饮水思源](http://bbs.sjtu.edu.cn/bbscon?board=WebDevelop&file=M.1371046203.A)。  

## 语言基础篇

### 0. this的指代
```js
function getThis(){
  return this;
}
console.log(getThis.call(1) === 1);
console.log(getThis.call('test') === 'test');
console.log(getThis.call() === undefined);
console.log(getThis.call(null) === null);
console.log(getThis.call(false) === false);
```
>> 在ECMA5严格模式中，this都会call和apply的第一个实参，哪怕传入的实参是原始值甚至是null或者undefined。在ECMA3和非严格模式中，传入的null和undefined会被替换成全局对象。  
>> 如果call和apply的第一个实参是原始类型，那么会先包装成对象。
>> 所以以上 **全部false**

### 1. bind & call
```js
function getThisAndArguments(){
  return {this: this, arguments: arguments};
}
console.log(getThisAndArguments.bind(1, 2).bind(3, 4).call(5, 6));
```
>> bind和call以及apply一样，如果第一个实参是原始类型，会包装成对象做为this的值。bind一旦绑定this之后就不可以再次绑定，所以this最终指向一个Number对象，其原始值1.  
>> arguments最终指向一个类似[2,4,6]的Arguments对象。  


### 2. 解释一下fn是做什么用的，以及它的声明语句为啥这么长。
```js
var fn = Function.prototype.call.bind(Array.prototype.forEach);
fn(document.querySelectorAll('*'), function(a){
  console.log(a);
});
```
>> 如果说只看代码声明还不知道fn是干什么的话，那么看了fn的调用就可以肯定它是干什么的。  
>> **不完全准确**地说，bind就是将一个函数作为一个对象的成员函数来执行。所以以上代码等价于以下代码：  
```js
// 该行代码是废话，可以忽略
Array.prototype.forEach.call = Function.prototype.call;
// 
Array.prototype.forEach.call(document.querySelectorAll('*'), function(a){
  console.log(a);
});
```
>> 看到上面的代码相比大家都明白了。  


### 3. Object & Function
结果分别是？解释一下
```js
console.log(Object instanceof Function);
console.log(Function instanceof Object);
```
>> 因为可以调用`new Object()`，所以Object肯定是函数。另外任何对象都是Object的实例，包括Object自身。  
以上答案经过Chrome验证是正确的。  
>> 

## 实战篇
以下题目是比较常见的场景，所以可能已经存在很好的JS库，请尽量不要参考。

### 4.
我们有一个 Node.js 爬虫，和一堆需要它去爬的连接，爬完后用log()记录内容，现在
请你改写这例子，控制爬虫并发请求数<=10。如果你能写出一个类似Python 
multiprocessing.Pool的可复用的东西，有加分噢。
```js
var urlArr = [...];
var url = urlArr[0];
crawl(url, callback(err, response){
    log(url, response);
});
```

### 5. 多层嵌套的优化
我们要用Nodejs查询若干次数据库，生成一个简单的博客页面，请你把如下例子尽可能改得优雅些。如果你能实现一个可复用的东西，有加分噢~
```js
db.getArticles(function(err, articles){
  if(err) return handle(err);
  db.getTags(function(err, tags){
    if(err) return handle(err);
    db.getCategories(function(err, categories){
      if(err) return handle(err);
      db.getComments(function(err, comments){
        if(err) return handle(err);
        render('index', {
          articles: articles,
          tags: tags,
          categories: categories,
          comments: comments
        });
      });
    });
  });
});
```

### 6.
由于历史的原因，我们有一个很傻的异步API：
```js
window.getSomeInfo = function(info){
  console.log(info); //info.url 是请求的url
};
var url = 'http://www.baidu.com/;
window.external.getSomeInfo(url); //异步执行，请求完成后会调用window.getSomeInfo()
```
如你所见，这个API没法支持两对及以上的 url->callback 组合。你能不能改写出一个稍微灵活点的wrapper？  


