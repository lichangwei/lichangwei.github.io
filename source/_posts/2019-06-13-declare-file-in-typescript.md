---
title: 集中地声明 TypeScript 类型
---

最近我将一个老项目改造成 TypeScript 项目，项目中有一些模型类型，这写类型在多个文件中都会使用，比如对用户的增删改查逻辑会分散在多个文件中，他们都会使用 User 这个类型，如果我把对 User 类型的生命放在任何一个文件中都会导致文件之间的互相引用，如果创建一个文件专门用来保存这个类型，又会导致文件更多并且这个文件内容略显单薄。后来就想到将所有的模型类型存放到一个单独的文件中，可以命名为`types.ts`，并确保该文件在`tsconfig.json`中的`includes`或`files`中，这样就可以文件中直接使用了。比如以下就是我的`types.ts`。
```ts
declare namespace GW {
    export type Image = {
        id: string;
        url: string;
        alt: string;
        description: string;
    };

    export type User = {
        id?: string;
        name: string;
        phone: string;
        email: string;
        username: string;
        password: string;
    };
}
```
在其他文件中直接使用，代码如下：
```ts
class PostAddStore{
    show = (post: GW.Post) => {
        // do something
    }
}
```

为了让敏感信息在传输中更加安全，在我的项目还用到`jsencrypt`对密码字段进行加密，而`jsencrypt`是没有类型声明的，因此我在`types.ts`文件中对它进行了声明，代码如下：
```ts
interface Window {
    JSEncrypt: new () => {
        setKey: (publicKey: string) => void;
        encrypt: (content: string) => string;
    };
}
```
这就表明`JSEncrypt`是一个构造函数，通过它创建的对象包含2个函数属性，分别是`setKey`和`encrypt`，以及这两个函数的类型声明。