---
title: 运行时指定 Webpack 的 publicPath
---
## 问题描述

一般情况下，我们在使用`Webpack`时，都会通过以下方式指定`publicPath`，它表示静态资源的访问地址。
```js
{
    output: {
        path: `${__dirname}/dest`,
        filename: 'app-[hash].js',
        publicPath: '',
    }
}
```

如果你需要把静态资源放在CDN上，则同样可行。
```js
{
    output: {
        path: `${__dirname}/dest`,
        filename: 'app-[hash].js',
        publicPath: 'https://cdn.example.com/',
    }
}
```

但是这样就把静态资源的地址写死了，而测试环境和生成环境的静态资源往往存在在不同的地方。OK，下面我就通过`__webpack_public_path__`变量来实现。

## 解决方法

1. 在`webpack`配置文件（比如`webpack.config.babel.js`）中删除`publicPath`的选项。
```js
output: {
    path: `${__dirname}/dest`,
    filename: 'app-[hash].js',
}
```

2. 在`html`模板文件中添加以下代码
```html
<script>window.webpackPublicPath = '<这里可以由后端负责填写>';</script>
```

3. 上个步骤中后端已经给我们提供了静态资源访问地址了，那么怎么应用到静态资源中呢？创建一个新文件，名字自己定，比如叫做`webpack.js`。
```
/* eslint-disable */
__webpack_public_path__ = window.webpackPublicPath;
```

4. 在App的入口文件的首行添加以下代码，其中`./webpack.js`就代表第三步创建的文件。
```
/* eslint-disable import/first */
import './webpack.js';
```

通过以上4个步骤即可实现有后端在代码运行时期指定`publicPath`了。