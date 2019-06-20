---
title: 在浏览器和 Node Server 之间加密敏感信息
---

我们的官网项目有个简单的后台管理系统，不开放注册，目前只有一个管理在用，功能比较少，也比较简单，主要就是发布文章，维护一些配置项等。所以在技术选型上我们选择对前端友好的`Node`，这样只要一个前端同事开发维护即可。后来在安全测试时，收到了一个Issue，说是在登录时没有将密码字段加密。关于这个问题，我认为通过`HTTPS`协议来发送请求已经足够安全了，但是我还是听从了安全测试同事的建议，对敏感信息做了加密处理。

对于加密分类想必大家都听说过，分为对称加密和非对称加密，对称加密有计算量小，加密速度快等特点，但是由于加密和解密双方使用同样的秘钥，安全性得不到保证。非对称加密有一对秘钥，公钥和私钥，用公钥加密后，只能用对应的私钥才能解密。非对称加密加解密速度比对称加密要慢很多，它不需要公开解密秘钥，即私钥，安全性很高。我们的项目是访问量很小的后台管理类应用，对性能要求也不高，所以我决定使用非对称加密。

`Node`语言自带加密模块[`crypto`](http://nodejs.cn/api/crypto.html)，在很早的版本中就提供了解密的API `crypto.privateDecrypt`，而在`10.12`版本中还添加了`crypto.generateKeyPair`新接口用来生成非对称加密的秘钥对。因此我打算在通过`npm scripts`生成秘钥对，前端在需要加密之前通过`AJAX`请求获取公钥，将敏感字段加密以后发给服务器。

关于如何生成秘钥对，我们可以参考`Node`官方文档中的示例代码并稍作修改，代码如下：
```js
const options = {
    modulusLength: 1024,
    publicKeyEncoding: {
        type: 'spki',
        format: 'pem',
    },
    privateKeyEncoding: {
        type: 'pkcs1',
        format: 'pem',
    },
};
crypto.generateKeyPair('rsa', options, (error, publicKey, privateKey) => {
    console.log(publicKey, privateKey);
});
```

这样我们就获得了一对公钥秘钥，我们可以把它们保存一个本地文件或数据库中。

在页面中需要引入[`jsencrypt`](https://www.npmjs.com/package/jsencrypt)，在需要发送敏感信息之前先发请求活动公钥，得到公钥以后进行加密，比如：
```js
const crypt = new window.JSEncrypt();
crypt.setKey(publicKey);
// 对密码字段进行加密
password = crypt.encrypt(password);
```

在服务器端得到加密后的密码字段以后，调用以下我封装好的方法即可获得原始密码字段。请注意`padding`字段的设置，否则可能会报错哦。
```ts
function decrypt(encrypted, privateKey) => {
    const buffer = crypto.privateDecrypt(
        {
            key: privateKey,
            padding: crypto.constants.RSA_PKCS1_PADDING,
        },
        Buffer.from(encrypted, 'base64'),
    );
    return buffer.toString('utf8');
}
```

到此我们就介绍完了如何在浏览器和`Node Server`之间传递敏感信息，是不是很简单呢？