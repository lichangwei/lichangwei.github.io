---
title: Nginx 指令 try_files 使用方法
---

我们公司的官网是使用`WordPress`做的，由市场部同事负责维护，运行了很多年，一直挺好。随着我们对官网页面特效追求不断提高，以及更加个性化，用 `WordPress`实现起来越来越麻烦，所以我们改为使用Node技术重建官网。具体技术方案参考[如何从`WordPress`中导入文章到数据库中](https://lichangwei.github.io/2019/03/05/import-posts-from-wordpress/)。理想是美好的，现实是骨感的，官网页面的更新方案迟迟没有出来，到目前为止，只有零星的几个页面，那么这些页面怎么上线呢？我们不想再把做好的页面放到`WordPress`中去，最好能够单独部署，并通过`nginx`的`80`端口给用户提供统一服务，让用户感知不到页面之间实现上的差别。逻辑如下：当用户请求过来时，首先查看某个目录下面是否有同名的静态文件，如果有，则直接返回，如果没有，则将请求转发给现有`wordpress`服务。

如果你对`Koa`比较熟悉，应该不难想到它和`Koa`的中间件功能类似，当用户请求过来时，首先进入第一个中间件，如果第一个中间能够处理该请求，则返回，否则第一个中间件通过调用`next`函数把该请求交给第二个中间件处理，以此类推。不同的是，`Koa`的中间件的模型和洋葱一样，外层中间件把请求交给内层中间件处理以后，它还有机会对处理结果做出修正，甚至完全忽视内层中间件的处理结果，虽然没人会这么做。
![](../../../../images/koa-middlewares.png)

闲话不多话，经过一些搜索我了解到`Nginx`的`try_files`指令可以实现这个逻辑。首先我们看一下[`Nginx`对`try_files`指令](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)的介绍。

```
语法: try_files file ... uri;
     try_files file ... =code;
默认值:	—
上下文:	server, location
```
按照指定顺序检查文件是否存在，并将找到的第一个文件做为请求处理结果。这个处理过程在当前上下文中执行，文件的路径是根据`root`或`alias`指令和`file`参数拼出来的，如果你在`file`参数后面添加一个斜线，比如`$uri/`也可以检查是否存在同名的目录。如果没有找到任何文件，将进行一个内部重定向到最后一个`uri`参数。最后一个参数可以指向一个命名地址（named location）或响应码（code）。
比如找到文件时返回404：
```
location / {
    try_files $uri $uri/index.html $uri.html =404;
}
```
找到文件时将请求转发给另一个服务：
```
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}
location @mongrel {
    proxy_pass http://mongrel;
}
```

看完了官方文档，再来看看文章开始提到的问题，答案就非常简单了：将老的`WordPress`应用的端口从80改成8080，并使用80端口响应我们整个服务
```
server {
    listen      80;
    root        /my/new/file/folder;
    try_files   $uri $uri.html @wordpress;

    location @wordpress {
        proxy_pass http://127.0.0.1:8080;
    }
}
```
以上配置看起来没有任何问题，但是访问时会被重定向到`http://localhost/<path>`上去，考虑到`proxy_pass`在转发时并不会转发客户端IP，Host等信息，猜测在老的`WordPress`应用检查了域名，发现不符时就进行了重定向。然后转发时添加了如下请求头，发现转发成功了。
```
server {
    listen      80;
    root        /my/new/file/folder;
    try_files   $uri $uri.html @wordpress;

    location @wordpress {
        proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;

        proxy_pass http://127.0.0.1:8080;
    }
}
```

以上，我们就通过`Nginx`的`try_files`指令将多个目录或站点合并起来并通过统一域名和端口提供服务。

参考资料
1. [`Nginx`的`try_files`指令介绍](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)
2. [详解proxy_pass、upstream与resolver](https://www.jianshu.com/p/5caa48664da5)