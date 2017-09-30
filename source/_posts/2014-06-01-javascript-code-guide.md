---
title: Javascript编码规范
---

## 1 变量

1. 如非必要，请不要使用全局变量；局部变量应该尽可能缩小其作用域。

1. 声明全局变量必须使用`window.`前缀；声明局部变量必须使用`var`关键字。比如  
```js
(function(){
  // 局部变量
  var config = {};
  // 全局变量
  window.global = {};
})();
```
1. 同时声明多个变量时，每个变量独立申明。
```js
// 好的做法
var var1 = 1;
var var2 = 'string';
var var3 = 3;
```
以前我们经常看到以下写法，以下做法会导致在末尾新增变量声明或者删除第一个变量声明时不方便。
```js
// 不好的做法
var var1 = 1,
    var2 = 'string',
    var3 = 3;
```
1. 鼓励延迟初始化
```js
```
1. 命名规范：命名应能够表达该变量的含义，首字母小写的驼峰格式。比如：
```js
var isInited = false;
```
例外：在循环体内可以使用i, j, k等无意义但是被广泛使用的变量。

## 2 常量
1. 命名规范：命名应能够表达该变量的含义，首字母小写的驼峰格式。比如`Math`的常量`PI`,`SQRT1_2`等等。

```js
var SECONDS_IN_A_DAY = 60 * 60 * 24;
```

## 3 语句

### 3.1 行
1. 语句必须以分号结束。
1. 每一行要控制在120字符以内。

### 3.2 空格
1. 数值操作符(如, +/-/*/% 等)两边留空;
2. 赋值操作符/等价判断符两边留一空格;
3. for 循环条件中, 分号后留一空格;
4. 变量声明语句, 数组值, 对象值及函数参数值中的逗号后留一空格;
5. 行尾不要有空格;
6. 逗号和冒号后一定要跟空格;
7. 点号前后不要出现空格;
8. 函数名末尾和左括号之间不要出现空格;

### 3.3 空行
1. 函数与函数声明之间，加一空行
1. 逻辑上独立的代码片段之间，加一空行。

### 3.4 缩进
1. 缩进使用2个空格

### 3.5 小括号
1. `if/else if/while/for`条件表达式必须有小括号;
1. 一元操作符(如 `delete`, `typeof`, `void`)或在某些关键词(如`return`, `throw`, `case`, `new`) 之后, 不要使用括号;

### 3.6 大括号
1. `if/else if/else/while/for`代码块中必须要有`{}`。
例外：函数体顶部出现的某中条件下函数直接返回的情况：`if(typeof arg !== 'string') return false;`，这时可以省略大括号并且将条件表达式和代码块写在同一行中。

### 3.7 内置对象
1. 禁止增加，删除或修改内置对象的方法，除非是polyfill（在旧浏览器上实现最新的规范）。

### 3.8 with
1. 禁止使用`with`，除非用于接续序列化字符串。

### 3.9 使用Array/Object直接量
1. 尽量使用使用Array/Object直接量，避免使用Array/Object构造器

### 3.10 字符串
1. 字符串应使用单引号。
1. 多行字符串应该使用`+`或者`Array.prototype.join`拼接字符串，避免使用`\`拼接。


## 4 注释
### 4.1 注释格式
1. 行级注释`// comment`主要用与代码行或代码片段
2. 块状注释`/* comment */`主要用于函数。
```js
/*
 * @desc 记忆函数，对于耗时的运算，运算一次以后就讲结果保存起来，下次就直接返回结果。
 * @param {function} func 运算非常耗时的函数。
 * @param {object} thisobj func的主体对象
 * @param {function} serialize 参数序列化函数，将参数序列化为一个字符串。
 */
function memorize(func, thisobj, serialize){}
```

### 4.2 注释书写原则
1. 在写注释之前，看看是否能够通过修改方法名，函数名让其变得见名知意，从而不需要注释。
1. 注释应该记录代码不能明显体现出来的思路或你对代码的评价。
1. 注释和代码一样，同样会占用屏幕空间，同样会分散阅读者的注意力，所以请保持言简意赅。
1. 修改代码的同时，请修改注释以保证注释的有效性。

### 4.3 注释标记
1. TODO 还没有完成的逻辑
1. FIXME 有错误的逻辑

## 5 函数
1. 函数行数不得超过40行，否则考虑拆分成小的函数。

1. 函数体内变量声明应尽可能集中在顶部。

1. 不要在块作用域中声明函数
不要写成:
```js
if (x) {
  function foo() {}
}
```
虽然有很多JS引擎都支持块做域内声明函数，但它不是ECMAScript规范(见[ECMA-262](http://www.ecma-international.org/publications/standards/Ecma-262.htm), 第13和14条)。各个浏览器的实现可能不兼容, 也可能与未来的ECMAScript草案相违背。ECMAScript只允许在脚本的根语句或函数中声明函数，如果确实需要在块中定义函数, 建议使用函数表达式来初始化变量:
```js
if (x) {
  var foo = function() {}
}
```

1. 命名规范：命名应能够表达该变量的含义，首字母小写的驼峰格式。比如：
```js
function returnTrue(){
  return true;
}
```

1. Getter/Setter 命名
这里单独说明Getter/Setter命名规范，一般情况下不需要使用Getter/Setter方法，直接暴露相应地对象属性即可。如确有必要，请使用getXxx和setXxx命名，对于boolean值，可以使用isXxx/hasXxx/canDoXxx等命名。


## 6 模块
1. 建议按照CMD或AMD规范实现模块，并使用seajs或requirejs等库加载或管理模块。
```js
define(function(require, exports, module){
  var $ = require('$');
  exports.init = function(){
    // do something
  };
});
```

## 7 文件
1. 命名规范：命名应能表达该文件的作用。全部小写，单词以连字符分隔，比如`category-manager.js`。
1. 编码规范：统一使用UTF-8（无BOM）格式。

## 8 HTML，CSS和Javascript
1. HTML中引用javascript文件时，建议去掉type属性。只有当script标签存放的不是javascript时才需要特别声明。
```html
<script src="path/to/js.js"></script>
<script>
  $(function(){
     // do something
  })
</script>
<script type="text/template">
	<div>{i18n.noData}</div>
</script>
```

##终极条款
1. 在修改别人的代码时，应该先学习别人的编码规范，并遵守该规范，如果有的话。

##参考
1. [Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
http://codeguide.bootcss.com/
