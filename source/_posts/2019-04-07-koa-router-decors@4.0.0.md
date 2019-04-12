---
title: koa-router-decors@4.0.0 功能介绍
---

上周[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)发布了它的第 4 个大版本 V4.0.0，这次更新带来了很多新的功能。从名字可以看出通过它可以使用装饰器编写 Koa 路由。确实如此，通过它，你不仅可以充分利用[koa-router](https://www.npmjs.com/package/koa-router)强大的功能，还可以忘记 koa-router 的存在，更加关注在核心业务逻辑。

下面我从两个方面介绍一下[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)，首先是基本用法，其次是高阶用法。

## 基本用法

[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)最基本的功能就是简化编写路由和加载路由的方法。下面我们来看一下它是如何简化的。

### 编写路由

[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)提供了 5 种名为`get`, `put`, `post`, `del` 和 `patch`的装饰器，表示该路由对应的 HTTP 方法，他们还接受一个字符串参数，表示该路由对应的 URL 地址。它从 koa-router 继承了一种能力，你可以编写类似`/user/:id`这种带有占位符的 URL，它通过
[path-to-regexp](https://www.npmjs.com/package/path-to-regexp)生成正则表达式来匹配某一类请求。

```ts
import { get, put, post, del, patch } from 'koa-router-decors';

class User {
    @get('/user/:id')
    async getUserById(ctx) {
        ctx.body = {};
    }

    @post('/user')
    async createUser(ctx) {
        ctx.body = {};
    }
}
```

通过以上步骤我们编写了路由文件，但是只有在启动应用时加载它们才能生效。比如使用了[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)的一个典型的应用入口文件`index.ts`是这样的：

```ts
import * as Koa from 'koa';
import { load } from 'koa-router-decors';

const app = new Koa();
app.use(load('/api', `${__dirname}/router`).routes());
app.listen(9100);
```

[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)还提供了一个`load`方法，我们通过它加载上面我们编写好的路由文件。它接受两个参数，第一个参数是`prefix`，表示所有 URL 的统一前缀，比如我们想让所有的 URL 以`api`打头，如果在每个路由文件中都写上`api`会很冗繁，并且想修改时要同时修改很多路由文件的很多处。第二个参数是路由文件所在文件夹地址，比如`` `${__dirname}/router` ``。它会同步加载该文件夹下所有路由文件中的所有路由，并返回一个 koa-router 实例。然后你就可以像使用 koa-router 一样使用它了。

这是[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)提供的最基本的功能，我们下面将介绍它的一些高阶功能。

## 高阶用法

### 1. 一个函数对应两个路由

比如在我们的官网项目中，通过标签查询文章的页面地址是`'/tag/:name/page/:page'`的，但是首页的页面地址是可以简化成`'/tag/:name/'`的，也就是说没有说第几页时就是第一页。这时候我们只需要将两个装饰器同时作用在一个路由上即可，是不是非常简单呢？

```ts
export default class Page {
    @get('/tag/:name/')
    @get('/tag/:name/page/:page')
    public async renderTagPage(ctx: Koa.Context) {
        await Page.getPost('tag', ctx);
    }
}
```

### 2. 处理 URL 前缀的特例

为了便于区分动态请求和和静态资源，我们一般给所有的动态请求都加上`/api`前缀，但是存在特例情况，个别请求地址因为第三方系统的限制，它并不是以`/api`打头的。怎么办呢？我们可以通过`get`等装饰器函数的第二个参数来指定，如下：

```ts
class User {
    @get('/method', { prefix: '' })
    async get(ctx: Koa.Context) {
        ctx.body = {};
    }
}
```

这样，上述文件所有生成的路由地址是`/method`，因为它指定了自己特殊的前缀，而不再使用`load`函数中指定的前缀。你可能会想到，如果特例情况较多呢，每次手动指定也很烦，没有关系，你可以将路由文件放置在两个文件夹中，两次调用`load`函数即可。

```ts
const app = new Koa();
// 可以多次调用`load`分别加载不同的目录
app.use(load('/api', `${__dirname}/router/api`).routes());
app.use(load('/web', `${__dirname}/router/web`).routes());
export default app.listen(9100);
```

### 3. 添加多个面向切面的中间件

在 Koa 中，想给所有路由添加一段面向切面的通用逻辑，比如添加日志，比如计算响应时间等，非常简单，通过`app.use`即可。但是如果只想给部分路由添加这种逻辑呢？如果还用`app.use`就根据 URL 等进行判断了，可能会用到正则表达式，可能会用到很多判断条件等，有没有简单一点方法呢？如果你使用[koa-router-decors](https://www.npmjs.com/package/koa-router-decors)就非常简单了。代码如下：

```ts
// 给该类中定义的所有路由添加 setResponseTime 中间件
@middlewares([setResponseTime])
class User {
    @get('/method')
    async get(ctx: Koa.Context) {
        ctx.body = {
            method: ctx.method,
        };
    }
}

// 以下就是一个给路由添加响应时间响应头的中间件
async function setResponseTime(ctx, next) {
    let date = new Date();
    await next();
    ctx.set('X-Response-Time', String(new Date().getTime() - date.getTime()));
}
```

如果只想给路由文件中某一个路由添加中间件可以这么做：

```ts
class User {
    // 给特定路由添加 setResponseTime 中间件
    @get('/method', { middlewares: [setResponseTime] })
    async get(ctx: Koa.Context) {
        ctx.body = {
            method: ctx.method,
        };
    }
}
```

以上就是[koa-router-decors@4.0.0](https://www.npmjs.com/package/koa-router-decors)所提供的功能了，欢迎使用，有问题请反馈给我，有建议也请联系我。
