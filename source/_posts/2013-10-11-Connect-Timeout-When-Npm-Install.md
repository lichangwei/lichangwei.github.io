---
title: npm install时连接超时的解决办法
---

今天在电脑上安装 node-inspector，用于在浏览器中调试 node。但是却遇到了如下连接超时错误。

```
sudo npm install -g node-inspector
npm http GET https://registry.npmjs.org/node-inspector
npm http GET https://registry.npmjs.org/node-inspector
npm http GET https://registry.npmjs.org/node-inspector
npm ERR! Error: connect ETIMEDOUT
npm ERR!     at errnoException (net.js:863:11)
...
```

想到可能是伟大的长城在作怪，所以首先想到的是加上代理，这里使用的是 Goagent。因为下载过程中使用的 https 协议，所以设置的应该是 https-proxy。设置方法如下：

```
npm config set https-proxy http://127.0.0.1:8087
```

依然失败，报错如下：

```
sudo npm install -g node-inspector
GET https://registry.npmjs.org/node-inspector
npm http GET https://registry.npmjs.org/node-inspector
npm http GET https://registry.npmjs.org/node-inspector
npm ERR! Error: UNABLE_TO_VERIFY_LEAF_SIGNATURE
npm ERR!     at SecurePair.<anonymous> (tls.js:1271:32)
```

根据以上错误，猜测应该是代理出现问题，网站证书无法验证。既然 https 协议走不通，那么是否可以使用 http 协议呢？有些网站都同时支持这两种协议，比如 cdn.staticfile.org，cdnjs.com 等 CDN 服务。向谷哥寻求帮忙才找到了答案，可以使用如下命令设置：

```
npm config set registry "http://registry.npmjs.org/"
```

还是失败，报错如下：

```
sudo npm install -g node-inspector
npm http GET http://registry.npmjs.org/node-inspector
npm http GET http://registry.npmjs.org/node-inspector
npm http GET http://registry.npmjs.org/node-inspector
npm ERR! Error: connect ETIMEDOUT
npm ERR!     at errnoException (net.js:863:11)
```

长城真是密不透风啊！为 http 设置代理：

```
npm config set proxy http://127.0.0.1:8087
```

然后终于看到亲切的`200`字眼。

```
sudo npm install -g node-inspector
npm http GET http://registry.npmjs.org/node-inspector
npm http 200 http://registry.npmjs.org/node-inspector
npm http GET http://registry.npmjs.org/node-inspector/-/node-inspector-0.5.0.tgz
npm http 200 http://registry.npmjs.org/node-inspector/-/node-inspector-0.5.0.tgz
npm http GET http://registry.npmjs.org/strong-data-uri
npm http GET http://registry.npmjs.org/async
```

以上就是今天遇到的通过`npm install`安装 node 包时遇到的问题以及解决过程。在遇到这些问题之前，还不知道`npm config`命令。所以在这里简单复习一下。
在控制台中输入`npm config`：得到如下结果：

```
npm config
npm ERR! Usage:
npm ERR! npm config set <key> <value>
npm ERR! npm config get [<key>]
npm ERR! npm config delete <key>
npm ERR! npm config list
npm ERR! npm config edit
npm ERR! npm set <key> <value>
npm ERR! npm get [<key>]
npm ERR! not ok code 0
```

`npm config`命令的使用方法都列出来了。如果想看到完全的信息，还是使用`npm help config`命令吧。这个命令会通过`vim`打开一个`npm config`命令的说明书，非常详尽，包括所有命令，各种类型的配置的优先级（比如命令行参数`--proxy http://server:port`优先级最高，文章中我们使用的方法是修改用户配置，优先级与命令行参数相比较低。）以及所有的配置选项（比如文章中我们修改了 proxy，https-proxy 和 registry.）。说明书篇幅很长，不再赘述。
