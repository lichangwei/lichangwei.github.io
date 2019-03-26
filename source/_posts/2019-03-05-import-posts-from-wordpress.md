---
title: 如何从`WordPress`中导入文章到数据库中
---

公司成立已经`10`年了，通过`WordPress`创建的官网也运行`10`年了。一直没有出现什么严重的问题，随着公司的成长，对官网的要求也在变高，后来有专业前端的加入实现页面效果，而`WordPress`对专业前端却没有那么友好了，后来我们决定抛弃`WordPress`对官网进行重构。考虑负责官网的人力少，项目比较简单，因此我们决定用`Node(Koa)`来实现一些动态请求，比如给用户发送手机验证码，收集用户表单数据等动态请求，以及后台管理系统；用`Pug + Less + Webpack + Browserify`编写静态页面等，通过阿里云`OSS`+`CDN`存储和分发图片等资源。这些都不是本文的重点，本文的重点是较少如何将现有官网中的文章迁移到新系统中。

下面我简单介绍一下`WordPress`，它是一个基于`PHP`和`MySQL`的开源内容管理系统，通常用于架设博客网站。通过插件和模板可以实现很多功能，是当今最流行的博客系统。

## 获取文章列表

在开始迁移老系统的文章这个任务之后，我首先找到官网系统的数据库，登录了以后发现文章表，但是为了安全起见，这个用户只能在服务器本机登录，我也不想添加用户或增加外部访问权限，所以我想通过`WordPress`公开`API`来获取文章列表，很幸运我搜索到 [公开 API：https://codex.wordpress.org/XML-RPC_WordPress_API](https://codex.wordpress.org/XML-RPC_WordPress_API)以及对应的[Node 软件包 wordpress](https://www.npmjs.com/package/wordpress)，这样我就做了一下尝试：

```node
var`WordPress`= require('wordpress');
var client = wordpress.createClient({
    url: 'http://www.domain.cn',
    username: '用户',
    password: '密码',
});

client.getPosts({ number: 10 }, function(error, posts) {
    console.log(posts);
});
```

很好，通过这个软件包我们已经能够获取`10`篇文章了，我们官网的文章大概有几百篇文章，因此我把`number`修改成`1000`，这样我就获取到所有文章了。但是我还发现这个接口给出的文章内容（`content`字段）不是最终的`HTML`文本，而包含了一些`WordPress`特殊标记，比如

```txt
[caption id="attachment_16488" align="aligncenter" width="721"]<img class="wp-image-16488 size-full" src="http://www.<domain>.cn/wp-content/uploads/2019/02/5s.jpg" alt="" width="721" height="3412" /> 5s[/caption]
```

所以我并没有能够获取文章的最终真实内容，我也搜索了这个问题，看了各种答案，最后还是需要通过网络爬虫的技术来获取文章内容。既然动用了网络爬虫了，我还需要上述的`wordpress`软件包吗？
我们也可以通过[`sitemap.xml`](https://baike.baidu.com/item/sitemap)文件来获取文章列表，然后爬取每篇文章的链接，从页面中获取文章内容，作者，标签，创建时间等，但是还是有一些信息并没有出现在这个页面，影响较大的就是文章摘要，和缩略图等。因此我决定继续使用`wordpress`软件包获取所有文章信息，对于文章内容则通过网络爬虫技术来获取。

## 爬取文章内容

怎么爬取文章内容呢？

我首先想到的是以前用过谷歌出品的 `HeadLess Chrome` 浏览器[`puppeteer`](https://www.npmjs.com/package/puppeteer)，但是考虑到安装比较复杂，并且有点杀鸡用牛刀的感觉，我就放弃了，通过在`npm`网站中搜索发现以前也了解过的[`jsdom`](https://www.npmjs.com/package/jsdom)，通过它提供的接口很容易就获取了文章内容，代码如下：

```js
const dom = await JSDOM.fromURL(post.link);
const document = dom.window.document;
const $content = document.querySelector('.entry-content');
const content = $content.innerHTML;
```

这样就把所有文章到导入到新的数据库中了。

## 生成文章摘要

很快发现了一个问题，大部分文章都没有摘要，可以猜想到`WordPress`是在渲染到页面时才根据文章内容生成摘要的，根据`WordPress`官网可以看到生成摘要的算法比较简单，就是从文章内容中截取前`90`个字符做为摘要。我还是希望把生成摘要的工作放在创建文章和修改文章，毕竟管理每天一班创建一篇文章，慢一点没有问题，而我们每天有成千上万的用户来查看文章。所以我需要根据文章内容生成摘要。

```js
const excerpt = post.excerpt || $content.textContent.trim().substring(0, 90);
```

注意在`jsdom`中元素是没有`innerText`属性的，需要使用`textContent`代替。这样摘要也生成了。文章已经从老系统迁移到新系统了。

## 转存图片资源

还有一点不要忘记，这些文章所引用的图片资源还存放在老系统上，我们需要将这些图片资源迁移到阿里云`OSS`上，并且修改文章中图片的地址。要操作阿里云`OSS`，当然首选阿里巴巴的一群小伙伴提供的`Node`软件包[`ali-oss`](https://www.npmjs.com/package/ali-oss)，它提供了上传图片的接口：`client.put(name, file)`，`name`就是文件名，而`file`是文章数据，可以是`String`, `Buffer`, `ReadStream`, `File`（浏览器），`Blob`（浏览器），比较好用的是可读流。代码如下

```js
http.get(url, async res => {
    if (res.statusCode !== 200) return onError();
    try {
        let path = require('url').parse(url).pathname;
        if (rename) path = rename(path);
        await oss.put(path, res);
    } catch (e) {
        onError();
    }
}).on('error', onError);
```

通过`Node`原生`API` `http.get`获取图片的可读流，请注意该接口既不是基于`Promise`的，也不是错误优先的，所以对其错误处理一定要小心。通过上述代码我们就可以将文章缩略图和文章内容中的图片上传到阿里云`OSS`中，生成基于`CDN`的图片地址，然后替换文章中相应字段即可。

## 爬虫相关问题

这样文章导入就没有业务上的问题了，但是还是出现一些错误，比如域名解析错误，可能是因为我们公司内部的域名服务器的问题，对于频繁的域名解析不能正确响应。我们没有详细了解，反正我频繁访问的域名只有一个，因此直接修改本机`hosts`文件。还有有些图片下载会失败，后来改为在每处理完一篇文章之后睡眠`2`秒之后再处理下一篇文章，避免服务器压力过大，或者触发反爬虫，反`DOS`攻击机制。代码也非常简单：

```js
// utils.ts
export default {
    sleep: ms => new Promise(r => setTimeout(r, ms)),
};
// import.ts
await utils.sleep(2000);
```

以上就是将基于`WordPress`的老官网中的文章迁移到新系统中遇到的问题，主要是通过`WordPress`的公开`API`获取文章列表，并通过爬虫相关技术获取文章内容。如果以上内容有误或你有任何建议，欢迎留言。
