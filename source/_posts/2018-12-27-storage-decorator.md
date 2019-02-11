---
title: 通过装饰器自动同步本地存储
---

为了让我们的产品更加人性化，很多时候我们需要记住用户的一些操作和设置。对于一些重要的设置，我们可以保存在服务器端，比如语言设置，优点是对于一个用户来说只需要设置一次，缺点是实现代价较大。还有大量不重要的设置，我们可以保存在浏览器端，通常通过`localStorage`本地存储来保存，优点是实现方便，缺点是对于一个用户来说，每次换浏览器都需要再设置一次。今天我们就来聊一聊怎么将用户操作和设置保存`localStorage`里。

<!--more-->

假设我们有一个日历页面，该页面提供了几个视图：天视图，周视图，月视图和年视图，用户可以通过按钮组来切换。当用户切换到某个视图以后我们就保存在`localStorage`里，以便当用户再次打开该页面时，我们能够自动切换到上次使用的视图。代码如下：

```js
onViewChange(view){
    this.view = view;
    // 首先我们需要在用户切换视图时将当前视图保存在`localStorage`里
    try{
        localStorage.setItem('calendar-view', view);
    }catch(e){
        // log error
    }
}

// 默认是周视图
this.view = 'week';
// 在用户再次进入该页面时切换到用户上次使用的视图
try {
    let view = localStorage.getItem('calendar-view');
    if(view){
        this.view = view;
    }
} catch (e) {
    // log error
}
```

功能完成了，代码似乎也不多，但是关键信息密度特别低，我们列出关键信息如下：局部变量`this.view`，本地存储键值`calendar-view`，默认值`week`。如果这样的代码只写一次不要紧，但是类似的需求很多，就很糟糕了。通过分析代码，我们首先看到的就是`localStorage`的接口在某些极端情况下可能报错，比如超过限额等。我们可以先封装一下这个接口。

```js
// storage.js
export default {
    setItem(key, value) {
        try {
            localStorage.setItem(key, value);
        } catch (e) {
            // log error
        }
    },
    getItem(key) {
        let value = null;
        try {
            value = localStorage.getItem('calendar-view');
        } catch (e) {
            // log error
        }
        return value;
    },
};
```

现在我们再来重构一下原来的文件

```js
import storage from './storage.js';

onViewChange(view){
    this.view = view;
    // 首先我们需要在用户切换视图时将当前视图保存在`localStorage`里
    storage.setItem('calendar-view', view);
}

// 在用户再次进入该页面时切换到用户上次使用的视图, 默认是周视图
this.view = storage.getItem('calendar-view') || 'week';
```

看起来已经很简单了，但是还是需要我们手动同步在`localStorage`里，有没有什么方法能让它自动同步到`localStorage`里？答案是肯定，接下里就是我们这篇文章的重点了。

[装饰器（Decorator）](https://tc39.github.io/proposal-decorators/)，可以作用在类，类的属性和方法上，来修改类的行为。一般用于记录日志，检查权限等通用的[切面](https://baike.baidu.com/item/AOP/1332219)的逻辑，可以减少代码侵入性，让你更加关注于处理核心业务逻辑，可以大大减少代码量。比如我们常用的`mobx`，就是通过装饰器来工作的。

```js
// store.js
export default class A {
    @observable
    field = 5;

    @action
    changeFieldValue(field) {
        this.field = field;
    }
}
// components.jsx
@observer
export default class MyComp extends React.Component{
    render(){}
}
```

作用在属性上的装饰器写法如下：

```js
/**
 * @param `target` 作用对象
 * @param `name` 属性名
 * @param `descriptor` 属性描述符
 * @return 如果返回属性描述符，则会调用`Object.defineProperty()`修改原有属性。
 */
function decorator(target, name, descriptor) {}
```

下面我们就来看看我写的用来同步数据到`localStorage`里的装饰器

```js
storage.sync = function(key) {
    return function(target, name, descriptor) {
        // 如果是普通属性
        if (descriptor.initializer) {
            //首选尝试从`localStorage`取出上次存储的值，如果有的话
            let value = storage.getItem(key);
            //如果没有`localStorage`保存该值，则使用该字段的默认值，比如`@observable field = 5`的默认值就是`5`
            if (_.isNil(value)) {
                value = descriptor.initializer();
            }
            // 返回一个新的属性描述符，以修改该字段的取值赋值行为
            return {
                // 每当赋值时，修改内部变量值，并将新值保存到`localStorage`中
                set: function(v) {
                    if (value === v) return;
                    value = v;
                    storage.setItem(key, v);
                },
                // 每当取值时，将内部变量返回出去，这里并不直接从`localStorage`取值是因为该操作性能较差
                get: function() {
                    return value;
                },
                enumerable: true,
                configurable: true,
            };
        }
        // 如果属性通过`get`和`set`定义，或者在使用当前装饰器之前还使用了其他装饰器
        else {
            // 首选尝试从`localStorage`取出上次存储的值，如果有的话
            let value = storage.getItem(key);
            // 如果`localStorage`已经保存该值，则调用原有属性描述符的`set`方法赋值
            if (!_.isNil(value)) {
                descriptor.set(value);
            }
            // 返回一个新的属性描述符，以修改该字段的取值赋值行为
            return {
                // 每当赋值时，将新值保存到`localStorage`中，并调用原有属性描述符的`set`方法赋值
                set: function(v) {
                    storage.setItem(key, v);
                    return descriptor.set(v);
                },
                // 每当取值时，调用原有属性描述符的`get`方法取值
                get: function() {
                    return descriptor.get();
                },
                enumerable: true,
                configurable: true,
            };
        }
    };
};
```

有了以上装饰器的实现以后，以后你再想同步某个变量到`localStorage`的时候，只需要这样写

```js
class A {
    @store.sync('local-storage-key')
    view = 'week';
}
```

是不是非常简单，没有任何冗余信息啊？你也快来试试吧。如果你有任何问题，欢迎联系我。
