---
title: Lodash函数以及常见用法
---

## 1 简介
[Lodash](https://github.com/lodash/lodash)是一款优秀的JavaScript工具库，里面包含了大量的工具函数。在2015年就成为[被依赖最多的JavaScript库](https://www.npmjs.com/browse/depended)，写这篇文档是最新版是4.17.4，适用于常见浏览器以及Node.js等。  

这里给出其[英文文档](https://lodash.com/docs)和[中文文档](http://lodash.think2011.net/)的链接。其中中文文档的版本较低，但是区别不大，可以参考帮助理解。

## 2 常见用法
在这部分我们介绍一些Lodash的常见的优雅的用法。主要是引起大家的学习兴趣，有更多优雅用法等待大家去发现。

### `_.get`获取一个嵌套很深的字段
```js
//config初始化为null，需要从服务器端获取权限数据
let config = null;
ajax.get(url, function(data){ // data = {basic: {delete: true}}
	config = data;
});
//使用原生JS获取是否有删除权限
let isDeletable = false;
if(config && config.basic){
	isDeletable = config.basic.delete || false;
}
//使用lodash获取是否有删除权限
let isDeletable = _.get(config, 'basic.delete', false);
```
对应地，可以通过`_.set({}, 'a.b.c', 1)`创建一个多级嵌套的对象。

### `_.map`获取数组中每个对象的特定字段，并形成一个新的数组
```js
//使用所有用户的idcard字段创建出一个数组
let users = [{idcard: '20160512', name: '张三'}, {idcard: '20160513', name: '李四'}];
//使用原生JS
let idcards = users.map(function(user){
	return user.idcard;
});
//使用lodash
let idcards = _.map(model, 'idcard');
```

### `_.pick`取出对象的部分字段形成一个新对象
```js
//在修改密码表单里，用户输入了如下字段并被封装到一个对象中
let form = {
	'password': '123456', //密码
	'repeatPassword': '123456', //重复密码
	'code': '5489' //验证码
};

//这里不直接使用delete删除字段，是因为该from对象与DOM进行了绑定。
//其中重复密码只用来在客户端校验，不需要发送给服务器
let data = {};
let fileds = ['password', 'code'];
for(var i = 0; i < fileds.length; i++){
	data[fields[i]] = form[fields[i]];
}

//使用lodash
let data = _.pick(form, 'password', 'code');
```
对应地，还有`_.omit`方法表示删除部分字段形成一个新对象。

### `_.random`获取一个随机值
```js
//获取[90, 100)之内的一个随机值
let min = 90;
let max = 100;
//使用原生JS
let random = Math.floor(Math.random() * (max - min)) + min;
//使用lodash
let random = _.random(min, max);
```
类似地，`_.sample(['a', 'b', 'c'])`可以从数组中随机取出一个项目。

### `_.clamp`将一个数字修改成区间中的一个值
```js
//使用原生JS
function applyRange(number){
	number = Math.max(number, this.props.min);
	number = Math.min(number, this.props.max);
	return number;
}
//使用lodash
let number = _.clamp(number, min, max);
```

### `_.once`确保一个函数只会执行一次
```js
//使用原生JS
let inited = false;
function init(){
	if(inited) return;
	// init code
}
//使用lodash
let init = _.once(function(){
	// init code
});
```

### `_.chain`链式操作
链接调用让代码更加整洁；避免了中间变量，避免了错误引用，让代码质量更有保证；`_.chain`还提供了延迟计算特性，在显式或隐式调用`value()`方法之前是不进行任何计算的，通过合并大大降低迭代次数。  
下面是`lodash`的官方文档中的一个例子。  
```js
var users = [
  { 'user': 'barney',  'age': 36 },
  { 'user': 'fred',    'age': 40 },
  { 'user': 'pebbles', 'age': 1 }
];

var youngest = _.chain(users)
  .sortBy('age')
  .map(function(chr) {
    return chr.user + ' is ' + chr.age;
  })
  .first()
  .value();
// → 'pebbles is 1'
```

## 3 模块
Lodash的工具函数很多，可以分为以下几类：数组（Array），集合（Collection），函数（Function），Lang（Lang），数学（Math），数字（Number），对象（Object），字符串（String），未分类工具函数（Util）。下面将会按类别介绍常见工具函数。

## 4 数组

### 获取子数组
|函数名|简介|
|---|---|
|[slice](https://lodash.com/docs/#slice)|获取元素第m-n(不包含)个元素|
|[tail](https://lodash.com/docs/#tail)|获取出第一个元素之外的其他元素|
|[initial](https://lodash.com/docs/#initial)|获取出最后一个元素之外的其他元素|
|[take](https://lodash.com/docs/#take)|从左侧开始获取任意数量的元素|
|[takeRight](https://lodash.com/docs/#takeRight)|从右侧开始获取任意数量的元素|
|[takeWhile](https://lodash.com/docs/#takeWhile)|从左侧开始获取任意数量的元素，直到断言返回假值|
|[takeRightWhile](https://lodash.com/docs/#takeRightWhile)|从右侧开始获取任意数量的元素，直到断言返回假值|
|[drop](https://lodash.com/docs/#drop)|丢掉前面几个元素，得到剩余元素|
|[dropWhile](https://lodash.com/docs/#dropWhile)|丢掉前面几个元素知道迭代器返回假值，得到剩余元素|
|[dropRight](https://lodash.com/docs/#dropRight)|丢掉后面几个元素，得到剩余元素|
|[dropRightWhile](https://lodash.com/docs/#dropRightWhile)|丢掉后面几个元素知道迭代器返回假值，得到剩余元素|


### 数组常见操作
|操作|不修改原数组|修改原数组|
|---|---|---|
|移除|[without](https://lodash.com/docs/#without)|[pull](https://lodash.com/docs/#pull)|
|相减|[difference](https://lodash.com/docs/#difference)|[pullAll](https://lodash.com/docs/#pullAll)|
|相减|[differenceBy](https://lodash.com/docs/#differenceBy)|[pullAllBy](https://lodash.com/docs/#pullAllBy)|
|相减|[differenceWith](https://lodash.com/docs/#differenceWith)|[pullAllWith](https://lodash.com/docs/#pullAllWith)|
|反转||[reverse](https://lodash.com/docs/#reverse)|
|裁剪|[at](https://lodash.com/docs/#at)|[pullAt](https://lodash.com/docs/#pullAt)|
|过滤|[filter](https://lodash.com/docs/#filter)|[remove](https://lodash.com/docs/#remove)|


### 数组常见操作变种函数 by, with

有些函数还可以稍微变化一下，接受不同的参数，提供更多灵活性。  

|作用|函数名|by|with|
|---|---|---|---|
|相减|[difference](https://lodash.com/docs/#difference)|[differenceBy](https://lodash.com/docs/#differenceBy)|[differenceWith](https://lodash.com/docs/#differenceWith)|
|交集|[intersection](https://lodash.com/docs/#intersection)|[intersectionBy](https://lodash.com/docs/#intersectionBy)|[intersectionWith](https://lodash.com/docs/#intersectionWith)|
|并集|[union](https://lodash.com/docs/#union)|[unionBy](https://lodash.com/docs/#unionBy)|[unionWith](https://lodash.com/docs/#unionWith)|
|异或|[xor](https://lodash.com/docs/#xor)|[xorBy](https://lodash.com/docs/#xorBy)|[xorWith](https://lodash.com/docs/#xorWith)|
|相减|[pullAll](https://lodash.com/docs/#pullAll)|[pullAllBy](https://lodash.com/docs/#pullAllBy)|[pullAllWith](https://lodash.com/docs/#pullAllWith)，跟[difference](https://lodash.com/docs/#difference)不同的是，[pullAll](https://lodash.com/docs/#pullAll)修改原数组|
|去重|[uniq](https://lodash.com/docs/#uniq)|[uniqBy](https://lodash.com/docs/#uniqBy)|[uniqWith](https://lodash.com/docs/#uniqWith)|
|去重|[sortedUniq](https://lodash.com/docs/#sortedUniq)|[sortedUniqBy](https://lodash.com/docs/#sortedUniqBy)|||


### 获取数组某个位置上的元素
|函数名|主要参数-返回值|简介|
|---|---|---|
|[head](https://lodash.com/docs/#head)|数组=>元素|返回数组的第一个元素，和[first](https://lodash.com/docs/#first)相同|
|[last](https://lodash.com/docs/#last)|数组=>元素|返回数组的最后一个元素，和[head](https://lodash.com/docs/#head)相反|
|[nth](https://lodash.com/docs/#nth)|数组=>元素|返回数组中某个位置上的元素|

### 检测元素在数组中的索引
|函数名|简介|
|---|---|
|[indexOf](https://lodash.com/docs/#indexOf)|获取元素在数组中的索引|
|[sortedIndexOf](https://lodash.com/docs/#sortedIndexOf)|和[indexOf](https://lodash.com/docs/#indexOf)功能一致，只是通过二分搜索方法|
|[lastIndexOf](https://lodash.com/docs/#lastIndexOf)|获取元素在数组中的索引，最后一次出现|
|[sortedLastIndexOf](https://lodash.com/docs/#sortedLastIndexOf)|和[lastIndexOf](https://lodash.com/docs/#lastIndexOf)功能一致，只是通过二分搜索方法|
|[findIndex](https://lodash.com/docs/#findIndex)|寻找元素位置|
|[findLastIndex](https://lodash.com/docs/#findLastIndex)|寻找元素位置，从后往前|

### 检测元素在插在有序数组的什么位置
|函数名|简介|
|---|---|
|[sortedIndex](https://lodash.com/docs/#sortedIndex)|通过二分搜索判断元素应该插在数组的哪个位置|
|[sortedIndexBy](https://lodash.com/docs/#sortedIndexBy)|同上，可以额外提供一个迭代器函数|
|[sortedLastIndex](https://lodash.com/docs/#sortedLastIndex)|和[sortedIndex](https://lodash.com/docs/#sortedIndex)类似，但是从右边开始|
|[sortedLastIndexBy](https://lodash.com/docs/#sortedLastIndexBy)|同上，可以额外提供一个迭代器函数|

### 将数组拍平
|函数名|主要参数-返回值|简介|
|---|---|---|
|[flatten](https://lodash.com/docs/#)|高维数组=>低维数组|将数组拍平|
|[flattenDeep](https://lodash.com/docs/#flattenDeep)|高维数组=>数组|将数组拍平|
|[flattenDepth](https://lodash.com/docs/#flattenDepth)|高维数组=>低维数组|将数组拍平|

### Zip
|函数名|主要参数-返回值|简介|
|---|---|---|
|[zip](https://lodash.com/docs/#zip)|多个数组=>二维数组|可以理解为二维数组的行列互换|
|[zipWith](https://lodash.com/docs/#zipWith)|多个数组=>数组|同上，但是可以自由处理行列互换后的数组中的每个数组元素|
|[zipObject](https://lodash.com/docs/#zipObject)|两个数组=>对象|把keys和values数组组成一个新对象|
|[zipObjectDeep](https://lodash.com/docs/#zipObjectDeep)|两个数组=>对象|同上，递归地处理属性名|

### 未分类函数
|函数名|主要参数-返回值|简介|
|---|---|----|
|[chunk](https://lodash.com/docs/#chunk)|数组=>二维数组|分段形成二维数组|
|[compact](https://lodash.com/docs/#compact)|数组=>数组|移除假值|
|[concat](https://lodash.com/docs/#concat)|多个数组=>数组|连接多个数组形成一个数组|
|[fill](https://lodash.com/docs/#fill)|数组=>数组|填充数组|
|[fromPairs](https://lodash.com/docs/#fromPairs)|二维数组=>对象|将键值数组变成对象。和[toPairs](https://lodash.com/docs/#toPairs)相反|
|[join](https://lodash.com/docs/#join)|数组=>字符串|拼接数组元素成一个字符串|

## 5 集合
为什么区分集合函数和数组函数？
集合函数不单单适用于数组，还适用于字符串，对象，类数组对象（比如Arguments，NodeList等）。字符串是字符的集合，对象是属性值的集合。类数组对象是通过“[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)”工作的，所以如果你传入一个拥有`length`字段并且值为数字的对象，这个对象会被当做一个数组处理。具体请参考[Underscore.js](http://underscorejs.org/#collections)文档。

范例：
```js
function printKeyVal(val, key){
	console.log(key, val);
}

//普通对象
_.each({'a': 1}, printKeyVal);
//打印结果
// a 1

//拥有值为数字的length字段
_.each({'a': 1, 'length': 2}, printKeyVal);
//打印结果
// 0 undefined
// 1 undefined
```
下面将分类介绍集合相关函数。

### 遍历
|函数名|简介|
|---|---|
|[each](https://lodash.com/docs/#each)|同[forEach](https://lodash.com/docs/#forEach)|
|[eachRight](https://lodash.com/docs/#eachRight)|同[forEachRight](https://lodash.com/docs/#forEachRight)|

### 排序
|函数名|简介|
|---|---|
|[sortBy](https://lodash.com/docs/#)|排序|
|[orderBy](https://lodash.com/docs/#orderBy)|同[sortBy](https://lodash.com/docs/#)，还可以指定正序倒序|
|[shuffle](https://lodash.com/docs/#shuffle)|返回一个打乱顺序的新数组|

### 过滤
|函数名|简介|
|---|---|
|[filter](https://lodash.com/docs/#filter)|创建一个新数组，包含了所有让断言为真的元素|
|[reject](https://lodash.com/docs/#reject)|创建一个新数组，包含了所有让断言为假的元素|
|[partition](https://lodash.com/docs/#partition)|根据断言真假将一个集合分成两个集合|


### Map之后再flatten
|函数名|简介|
|---|---|
|[flatMap](https://lodash.com/docs/#flatMap)|[map](https://lodash.com/docs/#map)之后再[flatten](https://lodash.com/docs/#flatten)|
|[flatMapDeep](https://lodash.com/docs/#flatMapDeep)|[map](https://lodash.com/docs/#map)之后再[flattenDeep](https://lodash.com/docs/#flattenDeep)|
|[flatMapDepth](https://lodash.com/docs/#flatMapDepth)|[map](https://lodash.com/docs/#map)之后再[flattenDepth](https://lodash.com/docs/#flattenDepth)|

### 寻找元素
|函数名|简介|
|---|---|
|[find](https://lodash.com/docs/#find)|找到第一个让断言为真的元素|
|[findLast](https://lodash.com/docs/#findLast)|同上，逆序|

### 随机取值
|函数名|简介|
|---|---|
|[sample](https://lodash.com/docs/#sample)|从集合中随机选出一个元素|
|[sampleSize](https://lodash.com/docs/#sampleSize)|从集合中随机选出n个元素|

### 迭代
|函数名|简介|
|---|---|
|[reduce](https://lodash.com/docs/#reduce)||
|[reduceRight](https://lodash.com/docs/#reduceRight)|-|

### 分组计数
|函数名|简介|
|---|---|
|[countBy](https://lodash.com/docs/#countBy)|返回一个对象，属性名是迭代器的返回值，属性值该返回值出现的次数|
|[groupBy](https://lodash.com/docs/#groupBy)|返回一个对象，属性名是迭代器的返回值，属性值是一个包含了相应元素的数组|

### 未分类
|函数名|简介|
|---|---|
|[keyBy](https://lodash.com/docs/#keyBy)|返回一个对象，属性名是迭代器的返回值，属性值是元素本身|
|[some](https://lodash.com/docs/#)|对于集合中的每个元素，是否至少其一返回真值|
|[every](https://lodash.com/docs/#every)|对于集合中的每个元素，是否都返回真值|
|[includes](https://lodash.com/docs/#includes)|判断元素是不是在数组中，判断某个值是不是某个对象的属性值，判断一个字符串是不是包含在另一个字符串中|
|[map](https://lodash.com/docs/#map)|对集合的数组|
|[invokeMap](https://lodash.com/docs/#invokeMap)|-|

## 6 对象
### 仅需要部分字段
|函数名|简介|
|---|---|
|[omit](https://lodash.com/docs/#omit)|通过忽略某些字段创建一个新对象|
|[omitBy](https://lodash.com/docs/#omitBy)||
|[pick](https://lodash.com/docs/#pick)|通过指定某些字段创建一个新对象|
|[pickBy](https://lodash.com/docs/#pickBy)|-|

### 合并对象
|函数名|简介|
|---|---|
|[assign](https://lodash.com/docs/#assign)|合并对象|
|[assignWith](https://lodash.com/docs/#assignWith)|有条件地合并对象|
|[extend](https://lodash.com/docs/#extend)|合并对象，包括原型链上的属性|
|[extendWith](https://lodash.com/docs/#extendWith)|有条件地合并对象，包括原型链上的属性|
|[assignIn](https://lodash.com/docs/#assignIn)|别名`extend`|
|[assignInWith](https://lodash.com/docs/#assignInWith)|别名`extendWith`|
|[defaults](https://lodash.com/docs/#defaults)|合并对象，将后面参数的属性付给第一个参数，如果第一个参数没有相应属性的话|
|[defaultsDeep](https://lodash.com/docs/#defaultsDeep)|递归地合并对象，将后面参数的属性付给第一个参数，如果第一个参数没有相应属性的话|
|[merge](https://lodash.com/docs/#merge)|递归地合并对象，将后面参数的属性付给第一个参数|
|[mergeWith](https://lodash.com/docs/#mergeWith)|同[merge](https://lodash.com/docs/#merge)，额外接受一个customizer参数|

### 键值列表
|函数名|简介|
|---|---|
|[keys](https://lodash.com/docs/#keys)|创建一个数组，包含对象中所有的属性|
|[keysIn](https://lodash.com/docs/#keysIn)|创建一个数组，包含对象中所有的属性（包含原型链上的）|
|[functions](https://lodash.com/docs/#functions)|创建一个数组，包含对象中所有值为函数的属性|
|[functionsIn](https://lodash.com/docs/#functionsIn)|创建一个数组，包含对象中所有值为函数的属性（包含原型链上的）|
|[values](https://lodash.com/docs/#values)|创建一个数组，包含对象中所有的属性值|
|[valuesIn](https://lodash.com/docs/#valuesIn)|创建一个数组，包含对象中所有的属性值（包含原型链上的）|

### 赋值取值
|函数名|简介|
|---|---|
|[at](https://lodash.com/docs/#at)|获取对象的一组属性路径的值，肯定不会报错|
|[get](https://lodash.com/docs/#get)|获取对象的某个属性路径的值，肯定不会报错|
|[result](https://lodash.com/docs/#result)|同[get](https://lodash.com/docs/#get)，但是如果属性值是函数的话，自动执行该函数|
|[set](https://lodash.com/docs/#set)|设置对象的某个属性路径的值|
|[setWith](https://lodash.com/docs/#setWith)|设置对象的某个属性路径的值，遇到不存在的中间对象，使用数组呢？还是使用对象呢？等等|
|[update](https://lodash.com/docs/#update)|同[set](https://lodash.com/docs/#set)，只是接受一个函数作为参数|
|[updateWith](https://lodash.com/docs/#updateWith)|同[setWith](https://lodash.com/docs/#setWith)，只是接受一个函数作为参数|
|[unset](https://lodash.com/docs/#unset)|删除对象的某个属性路径|
|[invoke](https://lodash.com/docs/#invoke)|调用对象中某个属性路径上的函数，肯定不会报错|

### 键值数组
|函数名|简介|
|---|---|
|[entries](https://lodash.com/docs/#entries)|{'a':1}=>[['a',1]]。别名`toPairs`|
|[entriesIn](https://lodash.com/docs/#entriesIn)|同上，但是包含原型链上的属性。别名`toPairsIn`|

### 键值变换
|函数名|简介|
|---|---|
|[mapKeys](https://lodash.com/docs/#mapKeys)|对对象中所有属性名做某种处理之后形成一个新对象|
|[mapValues](https://lodash.com/docs/#mapValues)|对对象中所有属性值做某种处理之后形成一个新对象|
|[invert](https://lodash.com/docs/#invert)|将对象中的属性名和属性值互换转成一个新对象|
|[invertBy](https://lodash.com/docs/#invertBy)|同[invert](https://lodash.com/docs/#invert)，但是转换以后的属性值是原属性值组成的数组|

### 键值遍历
|函数名|简介|
|---|---|
|[forIn](https://lodash.com/docs/#forIn)|遍历对象上的所有属性，包含原型链上的。|
|[forInRight](https://lodash.com/docs/#forInRight)|遍历对象上的所有属性，包含原型链上的。|
|[forOwn](https://lodash.com/docs/#forOwn)|遍历对象上的所有属性，不包含原型链上的。|
|[forOwnRight](https://lodash.com/docs/#forOwnRight)|遍历对象上的所有属性，不包含原型链上的。|

### 寻找属性
|函数名|简介|
|---|---|
|[findKey](https://lodash.com/docs/#findKey)|同[find](https://lodash.com/docs/#find)类似，但是匹配的是对象的属性值，返回的是对象的属性名|
|[findLastKey](https://lodash.com/docs/#findLastKey)|同[findKey](https://lodash.com/docs/#findKey)类似，但是匹配的是对象的属性值，返回的是对象的属性名|

### 判断属性是否存在
|函数名|简介|
|---|---|
|[has](https://lodash.com/docs/#has)|判断对象上是否拥有某个属性，不包含原型链上的|
|[hasIn](https://lodash.com/docs/#hasIn)|判断对象上是否拥有某个属性，包含原型链上的|

### 转换对象或数组
|函数名|简介|
|---|---|
|[transform](https://lodash.com/docs/#transform)|同[reduce](https://lodash.com/docs/#reduce)，但是其迭代器函数返回的是布尔值，如果返回false，则停止迭代|

### 创建新对象
|函数名|简介|
|---|---|
|[create](https://lodash.com/docs/#create)|创建一个对象，并指定其原型和属性|

## 7. 函数
### 修改参数
|函数名|简介|
|---|---|
|[ary](https://lodash.com/docs/#ary)|创建一个包裹函数，只将前n个参数传递给原函数。|
|[unary](https://lodash.com/docs/#unary)|创建一个包裹函数，只将第一个参数传递给原函数。|
|[flip](https://lodash.com/docs/#flip)|创建一个包裹函数，将参数逆序之后传递给原函数。|
|[rearg](https://lodash.com/docs/#rearg)|创建一个包裹函数，调整参数顺序之后在传递给原函数|
|[rest](https://lodash.com/docs/#rest)|创建一个包裹函数，将参数合成数组之后传递给原函数|
|[spread](https://lodash.com/docs/#spread)|创建一个包裹函数，将数组参数展开之后传给原函数，跟[rest](https://lodash.com/docs/#rest)相反|
|[overArgs](https://lodash.com/docs/#overArgs)|创建一个包裹函数，将参数做处理之后再传递给原函数。|

### 修改结果
|函数名|简介|
|---|---|
|[negate](https://lodash.com/docs/#negate)|创建一个包裹函数，返回原函数结果的非。|

### 缓存结果
|函数名|简介|
|---|---|
|[memoize](https://lodash.com/docs/#memoize)|创建一个包裹函数，会缓存计算结果|

### 降频调用
|函数名|简介|
|---|---|
|[debounce](https://lodash.com/docs/#debounce)||
|[throttle](https://lodash.com/docs/#throttle)||

### 延迟调用
|函数名|简介|
|---|---|
|[defer](https://lodash.com/docs/#defer)|类似`setTimeout(fn,0)`，可以指定参数|
|[delay](https://lodash.com/docs/#delay)|类似`setTimeout(fn,x)`，可以指定参数|

### 延迟调用
|函数名|简介|
|---|---|
|[once](https://lodash.com/docs/#once)|创建一个包裹函数，确保原函数只被执行一次。|
|[before](https://lodash.com/docs/#before)|创建一个包裹函数，确保原函数只被执行n次。|
|[after](https://lodash.com/docs/#after)|创建一个包裹函数，调用包裹函数时只有n次之后才会调用目标函数|

### 固定参数
|函数名|简介|
|---|---|
|[wrap](https://lodash.com/docs/#wrap)|创建一个包裹函数，固定原函数的第一个参数|
|[partial](https://lodash.com/docs/#partial)|创建一个包裹函数，固定原函数若干个参数|
|[partialRight](https://lodash.com/docs/#partialRight)|创建一个包裹函数，固定原函数若干个参数|
|[bind](https://lodash.com/docs/#bind)|创建一个包裹函数，固定原函数若干个参数，并指定this对象|
|[bindKey](https://lodash.com/docs/#bindKey)|和[bind](https://lodash.com/docs/#bind)功能类似，但是能够处理尚未创建或被重写的函数，有点事件代理的感觉。|
|[curry](https://lodash.com/docs/#curry)|创建一个包裹函数，可以传入任意数量的参数，如果参数不完整，则返回一个接受余下参数的新函数，否则，调用原函数获得计算结果。|
|[curryRight](https://lodash.com/docs/#curryRight)|同上，逆序|

## 8. 字符串
### 书写格式
|函数名|简介|
|---|---|
|[startCase](https://lodash.com/docs/#startCase)|每个单词首字母大写，多用于标题|
|[camelCase](https://lodash.com/docs/#camelCase)|小驼峰|
|[kebabCase](https://lodash.com/docs/#kebabCase)|小写连字符|
|[snakeCase](https://lodash.com/docs/#snakeCase)|小写下划线|
|[upperCase](https://lodash.com/docs/#upperCase)|大写加空格|
|[lowerCase](https://lodash.com/docs/#lowerCase)|小写加空格|

### 大写小写
|函数名|简介|
|---|---|
|[capitalize](https://lodash.com/docs/#capitalize)|首字母大写，其余小写|
|[upperFirst](https://lodash.com/docs/#upperFirst)|首字母大写，其余不变|
|[lowerFirst](https://lodash.com/docs/#lowerFirst)|首字母小写，其余不变|
|[toUpper](https://lodash.com/docs/#lowerFirst)|大写|
|[toLower](https://lodash.com/docs/#toLower)|小写|

### 打头结尾
|函数名|简介|
|---|---|
|[endsWith](https://lodash.com/docs/#endsWith)|是不是以特定字符串结尾|
|[startsWith](https://lodash.com/docs/#startsWith)|是不是以特定字符串打头|

### 转义
|函数名|简介|
|---|---|
|[escape](https://lodash.com/docs/#escape)|转义 &<>"'，与[unescape](https://lodash.com/docs/#unescape)相反|
|[escapeRegExp](https://lodash.com/docs/#escapeRegExp)|转义正则表达式中的特殊字符：^$.*+?()[]{}\||

### 补全抹掉
|函数名|简介|
|---|---|
|[pad](https://lodash.com/docs/#pad)|使用某个字符串将特定字符串扩充至指定长度，类似地还有[padStart](https://lodash.com/docs/#padStart)，[padEnd](https://lodash.com/docs/#padEnd)|
|[trim](https://lodash.com/docs/#trim)|去除字符串两边的特殊字符（默认为空格），类似地还有[trimStart](https://lodash.com/docs/#trimStart)，[trimEnd](https://lodash.com/docs/#trimEnd)|

### 未分类
|函数名|简介|
|---|---|
|[parseInt](https://lodash.com/docs/#parseInt)|转成整型|
|[repeat](https://lodash.com/docs/#repeat)|将某个字符串重复n遍|
|[replace](https://lodash.com/docs/#replace)|替换字符串|
|[split](https://lodash.com/docs/#split)|拆分字符串|
|[template](https://lodash.com/docs/#template)|简单模板引擎|
|[truncate](https://lodash.com/docs/#truncate)|截断字符串|
|[words](https://lodash.com/docs/#words)|将字符串拆分成单词，可以指定拆分模式|
|[deburr](https://lodash.com/docs/#deburr)|基本拉丁字母|

## 9. 数字
|函数名|主要参数-返回值|简介|
|---|---|----|
|[clamp](https://lodash.com/docs/#clamp)|数字=>数字|将数字限定在一个范围内|
|[inRange](https://lodash.com/docs/#inRange)|数字=>布尔|判断数字是否在某个区间里|
|[random](https://lodash.com/docs/#random)|区间=>数字|随机获取一个值，可以通过第三个参数指定是不是返回小数|

## 10. 数学
### 加减乘除
|函数名|主要参数-返回值|简介|
|---|---|----|
|[add](https://lodash.com/docs/#add)|两个数字 => 数字|返回两个数字的和|
|[subtract](https://lodash.com/docs/#subtract)|两个数字 => 数字|返回两个数字的差|
|[multiply](https://lodash.com/docs/#multiply)|两个数字 => 数字|返回两个数字的积|
|[divide](https://lodash.com/docs/#divide)|两个数字 => 数字|返回两个数字的商|

### 和，最大值，最小值，平均值
|函数名|主要参数-返回值|简介|
|---|---|----|
|[sum](https://lodash.com/docs/#sum)|数组 => 数字|返回数组中的各数字之和|
|[max](https://lodash.com/docs/#max)|数组 => 数字|返回数组中的最大值|
|[min](https://lodash.com/docs/#min)|数组 => 数字|返回数组中的最小值|
|[mean](https://lodash.com/docs/#mean)|数组 => 数字|返回数组中的平均值|

### 数字精度
|函数名|主要参数-返回值|简介|
|---|---|----|
|[ceil](https://lodash.com/docs/#ceil)|数字 => 数字|向上取整，可以指定精度|
|[floor](https://lodash.com/docs/#floor)|数字 => 数字|向下取整，可以指定精度|
|[round](https://lodash.com/docs/#round)|数字 => 数字|四舍五入取整，可以指定精度|

## 11. 语言
### 数值比较
|函数名|简介|
|---|---|
|[eq](https://lodash.com/docs/#eq)|等价于`===`|
|[isEqual](https://lodash.com/docs/#isEqual)|深度比较对象是否相等|
|[isEqualWith](https://lodash.com/docs/#isEqualWith)|深度比较对象是否相等，可以定义相等比较函数|
|[gt](https://lodash.com/docs/#gt)|大于|
|[lt](https://lodash.com/docs/#lt)|小于|
|[gte](https://lodash.com/docs/#gte)|大于等于|
|[lte](https://lodash.com/docs/#lte)|小于等于|

### 类型判断
|函数名|简介|
|---|---|
|[isArguments](https://lodash.com/docs/#isArguments)||
|[isArray](https://lodash.com/docs/#isArray)||
|[isArrayBuffer](https://lodash.com/docs/#isArrayBuffer)||
|[isArrayLike](https://lodash.com/docs/#isArrayLike)||
|[isArrayLikeObject](https://lodash.com/docs/#isArrayLikeObject)||
|[isBoolean](https://lodash.com/docs/#isBoolean)||
|[isBuffer](https://lodash.com/docs/#isBuffer)||
|[isDate](https://lodash.com/docs/#isDate)||
|[isElement](https://lodash.com/docs/#isElement)||
|[isEmpty](https://lodash.com/docs/#isEmpty)|判断是否有可遍历的属性|
|[isError](https://lodash.com/docs/#isError)|错误|
|[isFinite](https://lodash.com/docs/#isFinite)|是否是有限的数字，基于Number.isFinite|
|[isFunction](https://lodash.com/docs/#isFunction)||
|[isInteger](https://lodash.com/docs/#isInteger)||
|[isLength](https://lodash.com/docs/#isLength)||
|[isMap](https://lodash.com/docs/#isMap)||
|[isMatch](https://lodash.com/docs/#isMatch)||
|[isMatchWith](https://lodash.com/docs/#isMatchWith)||
|[isNaN](https://lodash.com/docs/#isNaN)||
|[isNative](https://lodash.com/docs/#isNative)|原生函数|
|[isNil](https://lodash.com/docs/#isNil)|等价于 `_.isNull(val)` &#124;&#124; `_.isUndefined(val)`|
|[isNull](https://lodash.com/docs/#isNull)||
|[isNumber](https://lodash.com/docs/#isNumber)||
|[isObject](https://lodash.com/docs/#isObject)||
|[isObjectLike](https://lodash.com/docs/#isObjectLike)||
|[isPlainObject](https://lodash.com/docs/#isPlainObject)||
|[isRegExp](https://lodash.com/docs/#isRegExp)||
|[isSafeInteger](https://lodash.com/docs/#isSafeInteger)||
|[isSet](https://lodash.com/docs/#)|isSet|
|[isString](https://lodash.com/docs/#isString)||
|[isSymbol](https://lodash.com/docs/#isSymbol)||
|[isTypedArray](https://lodash.com/docs/#isTypedArray)||
|[isUndefined](https://lodash.com/docs/#isUndefined)||
|[isWeakMap](https://lodash.com/docs/#isWeakMap)||
|[isWeakSet](https://lodash.com/docs/#isWeakSet)||

### 类型转换
|函数名|简介|
|---|---|
|[castArray](https://lodash.com/docs/#castArray)|强制转给数组|
|[toArray](https://lodash.com/docs/#toArray)|转成数组，对象调用Object.values，字符串转成字符数组|
|[toFinite](https://lodash.com/docs/#toFinite)||
|[toInteger](https://lodash.com/docs/#toInteger)||
|[toLength](https://lodash.com/docs/#toLength)||
|[toNumber](https://lodash.com/docs/#toNumber)||
|[toPlainObject](https://lodash.com/docs/#toPlainObject)||
|[toSafeInteger](https://lodash.com/docs/#toSafeInteger)||
|[toString](https://lodash.com/docs/#toString)|转成字符串，|

### 复制对象
|函数名|简介|
|---|---|
|[clone](https://lodash.com/docs/#clone)||
|[cloneDeep](https://lodash.com/docs/#cloneDeep)||
|[cloneDeepWith](https://lodash.com/docs/#cloneDeepWith)||
|[cloneWith](https://lodash.com/docs/#cloneWith)||

### 检测对象
|函数名|简介|
|---|---|
|[conformsTo](https://lodash.com/docs/#conformsTo)|判断一个对象的字段是否满足一些条件|

## 12. 工具
### 总是返回某个参数的函数
|函数名|简介|
|---|---|
|[constant](https://lodash.com/docs/#constant)|创建一个包裹函数，总是返回第一个参数|
|[nthArg](https://lodash.com/docs/#nthArg)|创建一个包裹函数，总是返回第n个参数|

### 总是返回某个特定值的函数
|函数名|简介|
|---|---|
|[noop](https://lodash.com/docs/#noop)|总是返回`undefined`的函数|
|[stubArray](https://lodash.com/docs/#stubArray)|总是返回空数组的函数|
|[stubObject](https://lodash.com/docs/#stubObject)|总是返回空对象的函数|
|[stubString](https://lodash.com/docs/#stubString)|总是返回空字符串的函数|
|[stubTrue](https://lodash.com/docs/#stubTrue)|总是返回`true`的函数|
|[stubFalse](https://lodash.com/docs/#stubFalse)|总是返回`false`的函数|
|[identity](https://lodash.com/docs/#identity)|总是返回第一个参数|

### 获取对象的属性值或者调用对象的函数
|函数名|简介|
|---|---|
|[method](https://lodash.com/docs/#method)|[_.invoke(object, path, [args])](https://lodash.com/docs/#invoke)预设`path`和`args`两个参数|
|[methodOf](https://lodash.com/docs/#methodOf)|[_.invoke(object, path, [args])](https://lodash.com/docs/#invoke)预设`object`和`args`两个参数|
|[property](https://lodash.com/docs/#property)|[_.get(object, path)](https://lodash.com/docs/#invoke)预设`path参数，不同的是缺少`defaultValue`参数|
|[propertyOf](https://lodash.com/docs/#propertyOf)|[_.get(object, path)](https://lodash.com/docs/#invoke)预设`object`参数，不同的是缺少`defaultValue`参数|

### 判断对象是否满足某些条件
|函数名|简介|
|---|---|
|[conforms](https://lodash.com/docs/#conforms)|创建一个包裹函数，判断一个对象的字段是否满足某个函数。`conforms`意思是遵守。|
|[matches](https://lodash.com/docs/#matches)|创建一个包裹函数，判断一个对象的字段是否等于某个值，使用[isEqual](https://lodash.com/docs/#isEqual)判断是否相等。跟[isMatch](https://lodash.com/docs/#isMatch)类似|
|[matchesProperty](https://lodash.com/docs/#matchesProperty)|创建一个包裹函数，判断一个对象特定字段是否等于某个值，使用[isEqual](https://lodash.com/docs/#isEqual)判断是否相等。|

### 把多个操作合成一个操作
|函数名|简介|
|---|---|
|[flow](https://lodash.com/docs/#flow)|把一组函数串起来形成一个新函数|
|[flowRight](https://lodash.com/docs/#flowRight)|同上，倒序|

### 批量进行多个操作
|函数名|简介|
|---|---|
|[over](https://lodash.com/docs/#over)|创建一个新函数，并将参数传递给预先指定的一组函数，并返回其结果|
|[overEvery](https://lodash.com/docs/#overEvery)|跟`over`类似，判断是不是所有函数都返回真值|
|[overSome](https://lodash.com/docs/#overSome)|跟`over`类似，判断是不是至少一个函数返回真值|

### 等差数列
|函数名|简介|
|---|---|
|[range](https://lodash.com/docs/#range)|生成等差数列，可以指定步长，步长可以是小数，也可以是负数|
|[rangeRight](https://lodash.com/docs/#rangeRight)|这个基本可以忽略，功能完成可以由[range](https://lodash.com/docs/#range)代替。|

### 其他未分类
|函数名|简介|
|---|---|
|[attempt](https://lodash.com/docs/#attempt)|使用try-catch包裹函数，如果出错返回错误对象|
|[bindAll](https://lodash.com/docs/#bindAll)|将一个对象的多个函数中的this固定为该对象|
|[cond](https://lodash.com/docs/#cond)|创建一个拥有复杂if-else的函数|
|[defaultTo](https://lodash.com/docs/#defaultTo)|如果第一个参数为NaN,null,undefined，则返回第二个参数，否则返回第一个参数|
|[iteratee](https://lodash.com/docs/#iteratee)|创建一个迭代函数|
|[noConflict](https://lodash.com/docs/#noConflict)|如果`_`被占用，可以使用该方法|
|[runInContext](https://lodash.com/docs/#runInContext)|创建一个`lodash`镜像对象，可以扩展修改该对象|
|[mixin](https://lodash.com/docs/#mixin)|给一个对象的原型添加属性或方法，一般配合[runInContext](https://lodash.com/docs/#runInContext)扩展`lodash`。|
|[times](https://lodash.com/docs/#times)|执行函数n次，传入参数为index|
|[toPath](https://lodash.com/docs/#toPath)|'a[0].b.c'=>['a','0','b','c']|
|[uniqueId](https://lodash.com/docs/#uniqueId)|生成唯一ID，可以指定前缀|

## 13. 链式

### 链式调用的好处
省略了中间变量，让代码更加简洁，更加安全。
链式调用可以优化成惰性求值（延迟计算），让代码更加高效。

### _(value)
创建一个经过lodash包装过后的对象会启用隐式链，直到调用了不支持链接调用的函数或者主动调用`value`方法解除链式调用。
作用类似于[chain](https://lodash.com/docs/#chain)

### lodash包装对象上的特殊函数
|函数名|简介|
|---|---|
|[tap](https://lodash.com/docs/#prototype-tap)|可以在链式调用中插入普通方法，直接修改中间结果，也可以仅仅是用于调试打印中间结果|
|[thru](https://lodash.com/docs/#prototype-thru)|同[tap](https://lodash.com/docs/#tap)，但是使用函数的返回值作为中间结果|
|[commit](https://lodash.com/docs/#prototype-commit)|立即执行链式调用中尚未进行的操作|
|[next](https://lodash.com/docs/#prototype-next)|获得包装对象的下一个值|
|[plant](https://lodash.com/docs/#prototype-plant)|复制一个链式调用，并传入初始值|
|[value](https://lodash.com/docs/#prototype-value)|结束链式调用，并计算结果。别名`valueOf`，`toJSON`|