---
title: 你们统统错了 - JavaScript 的连续赋值
---

相信很多人都看到过网上流传的各种面试题解析，其中一个便是`JavaScript`的连续赋值。首先要申明的是，这种题目不应该作为普通前端岗位的面试题目存在，也更不应该编写这样的代码。

网上有很多文章都在解析这到题目，题目基本一样，我在下面贴出了最常见的一个。

```js
var a = { n: 1 };
var b = a;
a.x = a = { n: 2 };
console.log(a.x);
console.log(b.x);
```

答案可能很难回答，但是要想知道答案却很简单，只要在控制台执行一下即可：`undefined`, `{n:2}`. 对于这个答案网上有很多推测，但是好像统统错误。我也曾对这个答案做过各种推断，尝试解释这个答案，尝试自圆其说。但是很快我意识到这样做是错误的，我们应该阅读`ECMAScript`规范，应该阅读`V8`引擎的源码。规范难读，`C++`源码难看，还能怎么做呢？OK，下面我通过`Proxy`来揭露连续赋值的过程。首先我们改造一下代码，将`a`改成一个代理对象，这样对`a`的修改将一目了然。

```js
// 初始化代码
var a = new Proxy(
    { n: 1 },
    {
        get: function(target, prop) {
            console.log('get', prop, target[prop]);
            return target[prop];
        },
        set: function(target, prop, value) {
            console.log('set', prop, value);
            target[prop] = value;
        },
    }
);
var b = a;
```

现在我们执行一下代码

```js
a.x = a.y = 1;
```

控制台打印如下：

```
set y 1
set x 1
```

由此可以验证，如我们以前所被告知的那样的，连续赋值操作是从右往左执行的。我们再来执行代码

```js
// 为了消除前一步代码执行的，请在这里添加上面的初始化代码
a.x.y = a.y.x = 1;
```

控制台打印如下：

```
get x undefined
get y undefined
Uncaught TypeError: Cannot set property 'y' of undefined
```

注意，根据以上结果我们判断，`V8`引擎首先获取`a.x`的值，虽然是`undefined`也不直接报错，继续获取`a.y`的值，同样是`undefined`也不报错，然后执行赋值操作，给`a.y`即`undefined`添加`x`属性时报错，赋值操作中断。

现在再来解释连续赋值操作，就很容易理解了吧。
