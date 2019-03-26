---
title: 如何快速找到 Nginx 配置文件？
---

我虽是前端工程师，也经常会去 Linux 服务器上修改 Nginx 配置文件并且重启，但是那么多的服务器，每个服务器上安装 Nginx 的版本和方法也不尽相同，导致 Nginx 配置文件的位置并不相同，那怎么才能快速地找到 Nginx 配置文件呢？这里介绍一个简单的方法。

## 如何快速找到 Nginx 配置文件

不知道你是否还记得检查 Nginx 配置文件是否合法的那个命令吗？对的，就是`nginx -t`。如果通过检查，你会看到类似以下信息，

```bash
nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

如果没有通过检查，就有各种可能错误，比如以下就是一种可能：

```bash
nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: [emerg] open() "/aa/aa.log" failed (2: No such file or directory)
nginx: configuration file /usr/local/etc/nginx/nginx.conf test failed
```

你可能已经注意到不管配置文件是否正确，`nginx -t`命令都会告诉你 Nginx 配置文件在哪里，所以通过执行`nginx -t`命令可以帮我们找到 Nginx 配置文件。

## 如何快速找到任何我想要的文件

你可能会说，上面的方法不具有通用性，如果我想找到其他文件就没用了。的确是这样，不过我接下来介绍一个命令，它可以帮你快速找到任意一个文件。

我们要安装命令是`locate`命令，它和`find -name`作用类似，但是速度更快，因为不会检索文件目录，而是去事先建立好的数据库里查询。这个数据库每天更新一次，因次如果你的文件是刚刚创建的，是不能立即搜索到的，需要手动执行`updatedb`命令才行。

说了那么多，我们来看看怎么安装吧？

```bash
yum install mlocate
```

安装好了，就可以用起来了，该命令常见参数如下：
-b：只匹配 Base Name，比如`locate -b '\passwd'`；
-i：忽略大小写，比如`locate -i PASSWD`
-r：正则表达式。

它也有配置文件，一般位于`/etc/updatedb.conf`，通过它可以配置，哪些后缀的文件会被忽略，哪些目录下的文件会被忽略等。必要时可以修改该配置文件。
