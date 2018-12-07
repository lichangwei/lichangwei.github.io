---
title: 【Node系列】回调地狱和异步编程
---

## 回调地狱

`Node`中有大量的异步 IO 操作，被封装成基于回调的函数，遇到复杂的业务逻辑很容易形成多级缩进的代码，在左侧形成一个由空格（或 Tab）组成的三角形，代码变得非常难读，被称为回调地狱。
![image_1bjrp088i1fepi1o1klpdarjs1m.png-238kB][1]

```js
const fs = require('fs');
const _ = require('lodash');

function compose() {
    //读取页面模板
    fs.readFile('sample.template.html', 'utf-8', function(error, template) {
        if (error) throw error;
        //读取用户数据
        fs.readFile('sample.data.user.js', 'utf-8', function(error, user) {
            if (error) throw error;
            //读取公司数据
            fs.readFile('sample.data.company.js', 'utf-8', function(error, company) {
                if (error) throw error;
                user = JSON.parse(user);
                company = JSON.parse(company);
                //组装页面
                let data = { user, company };
                let html = _.template(template)(data);
                console.log(html);
            });
        });
    });
}

compose();
```

下面我们来看一下，怎么解决回调地狱问题。

## 解决方案

1. 函数拆解并使用第三方异步库
2. Promise
3. 生成器函数
4. 终极大招：async/await

## 函数拆解并使用第三方异步库

```
const fs = require('fs');
const _ = require('lodash');
const async = require('async');

function readTemplate(callback){
	//读取页面模板
	fs.readFile('sample.template.html', 'utf-8', callback);
}

function readUserData(callback){
	//读取用户数据
	fs.readFile('sample.data.user.js', 'utf-8', callback);
}

function readCompanyData(callback){
	//读取公司数据
	fs.readFile('sample.data.company.js', 'utf-8', callback);
}

function compose(template, user, company){
	user = JSON.parse(user);
	company = JSON.parse(company);
	//组装页面
	let data = {user, company};
	let html = _.template(template)(data);
	console.log(html);
}

async.series([
	readTemplate,
	readUserData,
	readCompanyData
], function(error, results){
	compose(results[0], results[1], results[2]);
});
```

### 使用第三方库

1. 多个逻辑单元被分成独立的函数。
2. 每个函数有了有意义的名称，更加易读。
3. 依赖第三方异步类库解决回调地狱问题。

## Promise

### Promise 简介

![image_1bjrhnves1ocn183b1d3n11thb119.png-17.3kB][3]

### Promise 成为 JavaScript API 的基石

[从 Node 6.X 开始内置 Promise](http://node.green/# ES2015-built-ins-Promise). JavaScript 相关生态中更多的 API 都开始基于 Promise 实现。比如下面的两段代码。

Battery API，提供了有关系统充电级别的信息并提供了通过电池等级或者充电状态的改变提醒用户的事件。 这个可以在设备电量低的时候调整应用的资源使用状态，或者在电池用尽前保存应用中的修改以防数据丢失。

```js
//获取设备电池相关数据
navigator.getBattery().then(function(battery) {
    console.log(battery);
    // {
    // 	charging: true
    // 	chargingTime: 0
    // 	dischargingTime: Infinity
    // 	level: 1
    // 	onchargingchange: null
    // 	onchargingtimechange: null
    // 	ondischargingtimechange: null
    // 	onlevelchange: null
    // }
});
```

Fetch API 提供了一个 JavaScript 接口，用于访问和操纵 HTTP 管道的部分，例如请求和响应。它还提供了一个全局 fetch()方法，该方法提供了一种简单，合乎逻辑的方式来跨网络异步获取资源

```js
fetch('flowers.jpg')
    .then(function(response) {
        return response.blob();
    })
    .then(function(blob) {
        let objectURL = URL.createObjectURL(blob);
        document.querySelector('img').src = objectURL;
    });
```

### Promise 类方法简介

```js
Promise.resolve(1);
//等价于以下代码
new Promise(function(resolve, reject) {
    resolve(1);
});
```

```js
Promise.reject(1);
//等价于以下代码
new Promise(function(resolve, reject) {
    reject(1);
});
```

`Promise.all`：将多个 Promise 实例，包装成一个新的 Promise 实例。有一个 Promise 对象状态改变成`rejected`，新的 Promise 实例的状态就改变成`rejected`，否则等所有状态改变成`fulfilled`，新的 Promise 实例的状态就改变成`fulfilled`。

`Promise.race`：将多个 Promise 实例，包装成一个新的 Promise 实例。有一个 Promise 对象状态改变，新的 Promise 实例的状态就改变，新的 Promise 实例的状态就是第一个改变状态的 Promise 实例的状态。

### Promise 实例方法

Promise.prototype.then
Promise.prototype.catch

### 使用 util.promisify 转成基于 Promise 的函数

```js
const { promisify } = require('util');
const read = promisify(require('fs').readFile);

read(path, 'utf-8').then(
    function(txt) {
        console.log(txt);
    },
    function(err) {
        console.error(err);
    }
);

let date = new Date();
promisify(setTimeout)(10000).then(function() {
    console.log(new Date() - date);
});
```

### 自定义基于 Promise 的函数

使用`fn[util.promisify.custom]`来定义基于 Promise 的接口。

```
const util = require('util');

function foo() {
    return 'abc';
}
async function fooAsync() {
    return 'abc';
}
foo[util.promisify.custom] = fooAsync;

console.log(util.promisify(foo) === fooAsync); // true
```

### 使用 Promise 解决回调问题

```js
//使用`util.promisify`转成基于Promise的接口
const { promisify } = require('util');
const read = promisify(require('fs').readFile);
const _ = require('lodash');

function usePromise() {
    var template, user, company;
    //读取页面模板
    read('sample.template.html', 'utf-8')
        .then(t => (template = t))
        .then(function() {
            //这里必须return，否则下面的一个 then 不等待 user 数据
            //读取用户数据
            return read('sample.data.user.js', 'utf-8').then(u => (user = u));
        })
        .then(() => {
            //读取公司数据
            return read('sample.data.company.js', 'utf-8').then(c => (company = c));
        })
        .then(() => {
            console.log(template, user, company);
        });
}
//使用Promise.all
function usePromiseAll() {
    Promise.all([
        read('sample.template.html', 'utf-8'),
        read('sample.data.user.js', 'utf-8'),
        read('sample.data.company.js', 'utf-8'),
    ]).then(([template, user, company]) => {
        compose(
            template,
            user,
            company
        );
    });
}
//组装页面
function compose(template, user, company) {
    let data = {
        user: JSON.parse(user),
        company: JSON.parse(company),
    };
    let html = _.template(template)(data);
    console.log(html);
}

usePromise();
usePromiseAll();
```

## 通过生成器函数

生成器函数是一个状态机，封装了多个内部状态。还是一个遍历器生成函数，返回遍历器对象，可以依次遍历生成器函数内部的每一个状态。

### 生成器函数执行器

[co](https://github.com/tj/co)是一个基于生成器函数的流程控制工作，可用于 Node.js 和浏览器。它可以通过 Promise 让你的非阻塞代码以一种漂亮的方式呈现。

```js
let print = val => console.log(val);

co(function*() {
    return yield Promise.resolve(true);
}).then(print);
```

```js
var fn = co.wrap(function*(val) {
    return yield Promise.resolve(val);
});

fn(true).then(print);
```

### 使用生成器函数解决回调地狱问题

```js
//使用`util.promisify`转成基于Promise的接口
const { promisify } = require('util');
const read = promisify(require('fs').readFile);
const _ = require('lodash');
const co = require('co');

//组装页面
function compose(template, user, company) {
    let data = {
        user: JSON.parse(user),
        company: JSON.parse(company),
    };
    let html = _.template(template)(data);
    console.log(html);
}

let useGenerator = co.wrap(function*() {
    let template = yield read('sample.template.html', 'utf-8');
    let user = yield read('sample.data.user.js', 'utf-8');
    let company = yield read('sample.data.company.js', 'utf-8');
    compose(
        template,
        user,
        company
    );
});

useGenerator();
```

## 终极大招：async/await

### 生成器函数和 async 函数比较

使用生成器函数

```js
const { promisify } = require('util');
let read = promisify(require('fs').readFile);

var fn = function*() {
    var f1 = yield read('/etc/fstab');
    var f2 = yield read('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
};
require('co')(fn);
```

使用`async`函数

```js
var fn = async function() {
    var f1 = await read('/etc/fstab');
    var f2 = await read('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
};
```

`async`函数和生成器函数非常相似，基本上就是`*`换成`async`，`yield`换成了`await`。

### async 函数的优点

1. 内置了执行器
   生成器函数的执行必须靠执行器，所以才有了 co 模块，而 async 函数自带执行器。也就是说，async 函数的执行，与普通函数一模一样，只要一行。
2. 更好的语义
   async 和 await，比起星号和 yield，语义更清楚了。async 表示函数里有异步操作，await 表示紧跟在后面的表达式需要等待结果。
3. 更广的适用性。
   co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。
4. 返回值是 Promise。
   async 函数的返回值是 Promise 对象，这比生成器函数的返回值是 Iterator 对象方便多了。你可以用 then 方法指定下一步的操作。

进一步说，async 函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而 await 命令就是内部 then 命令的语法糖。

### 使用 async 函数解决回调地狱问题

```js
const { promisify } = require('util');
const read = promisify(require('fs').readFile);
const _ = require('lodash');

//组装页面
function compose(template, user, company) {
    let data = {
        user: JSON.parse(user),
        company: JSON.parse(company),
    };
    let html = _.template(template)(data);
    console.log(html);
}

let useAsync = async function() {
    let template = await read('sample.template.html', 'utf-8');
    let user = await read('sample.data.user.js', 'utf-8');
    let company = await read('sample.data.company.js', 'utf-8');
    compose(
        template,
        user,
        company
    );
};

useAsync();
```

## 总结

1. 回调地狱问题可以通过第三方异步类库，Promise，生成器函数和 async 函数等方式解决。
2. Promise 是 ECMAScript 中异步 API 的基石，需要重点掌握。
3. async 函数解决异步问题更加优雅，推荐在 Node 中使用。

## 参考资料

1. [Node.js 8: util.promisify()](http://2ality.com/2017/05/util-promisify.html)
2. [ECMAScript 6 入门](http://es6.ruanyifeng.com/) 阮一峰
3. [co](https://github.com/tj/co) 生成器函数执行器

[1]: http://static.zybuluo.com/lichangwei/4dcltssgjnp2b2293nei6a3g/image_1bjrp088i1fepi1o1klpdarjs1m.png
[2]: http://static.zybuluo.com/lichangwei/7dr9ts0gz0jpae4gd7d6dbrm/image_1bjrp2s8f3nf2np1k8p1u1f1vh513.png
[3]: http://static.zybuluo.com/lichangwei/9rx1tdnqg4dk942otjej824b/image_1bjrhnves1ocn183b1d3n11thb119.png
