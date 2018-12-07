---
title: 在客户端高亮关键字
---

## 背景介绍

在某个项目中遇到这样的需求，用户输入几个关键字，然后根据关键字去后端搜索相关记录。后端返回记录中但是没有高亮这些关键字，这就需要前端来做这件事。按道理，高亮应该由后端完成，因为后端如何解析这些关键字，也知道如何去匹配，前端不知道也不应该知道后端为什么返回这些结果，英语中名词有复数形式，动词有现在式，过去式，过去完成式等，其他诸如德语，葡萄牙语有没有词性变换更是不得而知。后端依赖强大的知识库可以处理词性变换甚至近义词都是可能的，前端不可能完成工作，即便可能，也远比后来来处理高亮要复杂的多。但是这就是客户需求，我们还是要实现。

整个流程：获取用户输入，比如 'search by keyword'，发送 JSONP 请求获取后端搜索到的记录。在关键的几个字段中，搜索‘search’，‘by’和‘keyword’这三个关键字，并在单词两边分别加上`<span class="highlight">`和`</span>`，给`.highlight`定义一个合适的样式，比如粗体等，就完成了高亮的效果。

## 步骤

第一步，首先把关键字用空白符分隔开，得到一个一个的关键字。

```js
var keywords = keyword.split(/\s+/);
```

第二步，将第一步中得到的字符串数组，转变成正则数组。

```js
keywords.map(function(it) {
    // g replace all
    // i case-insensitive
    return new RegExp(it, 'gi');
});
```

RegExp 的第一个参数是传 it 呢，还是`'\b' + it + '\b'`呢？如果是前者，关键字`a`可以匹配`take`中的`a`。如果是后者，`teach`不可以匹配`teaches`中的`teach`。都不完美，这也是在文章开头提及的不适合前端做高亮的原因。  
如果你够细心，你应该已经感觉到隐隐有些不妥，因为在通过 RegExp 创建正则表达式时，第一个参数不能包含正则中预定义的一些字符，除非在前面加上了一个反斜线，比如`[?`就不可以，而`\[\?`才是真正地匹配`[?`。所以在这之前，我们添加一步：

第〇步：给所有的正则预定义字符之前添加一个反斜线（backslash）。找出所有的预定义字符`char`并替换为`'\\' + char`。通过强大的`String.prototype.replace`方法很方便的完成。

```js
keyword = keyword.replace(/[\\\{\}\[\]\(\)\*\+\?\^\$\,\-\=\!]/g, function(char) {
    return '\\' + char;
});
```

需要特别说的是，还有一处可以改进。在前面的一片文章[【正则表达式】使用逗号将数字三位三位地分开](./2013-04-15-Grouping-Numbers-with-Comma.md)中，我们使用了零宽断言。它匹配了这样一个位置（是为零宽），它后面的字符满足一定条件（是为断言）。在这里，我们可以使用正则表达式匹配预定义字符前面的位置，将该位置替换为`'\\'`。

```js
keyword = keyword.replace(/(?=[\\\{\}\[\]\(\)\*\+\?\^\$\-\=\!])/g, '\\');
```

换了一种思路，顿时感觉清爽多了。

第三步，遍历正则数组，然后替换所有的关键字。

```js
keywords.forEach(function(it) {
    string = string.replace(it, function(val) {
        return '<span class="highlight">' + val + '</span>';
    });
});
```

如果你足够细心或者以前遇到过类似问题，你可能已经知道，这一步中隐藏着一些问题，就是如果用户在一次搜索中，输入了两个相同的关键字，比如`take take`，那么匹配以后可能会出现`'<span class="highlight"><span class="highlight">take</span></span>'`这种情况，不会出错，只是浏览器需要多创建一些 DOM 元素。更甚的是，当用户输入多个关键字以后，第一次替换时会引入`'<span class="highlight">'`和`'</span>'`片段，这两段字符是不应被匹配并替换的。比如替换字符是`'programer'`，关键字是`'a span'`，
匹配关键字`a`以后，变成

```html
progr<span class="highlight">a</span>mer
```

匹配关键字`span`以后：

```html
progr<<span class="highlight">span</span> class="highlight">a</<span class="highlight">span</span>>mer
```

完全乱套了。  
为了保证以上`span`两端字符不会被匹配并替换，有一种做法是仅仅记住需要插入`span`的位置，并延迟替换。如果记住位置，考虑使用一种基本不会被用到的字符，比如 0X2611(☑)以及 0X2612（☒）。

```js
keywords.forEach(function(it) {
    string = string.replace(it, function(val) {
        return String.fromCharCode(0x2611) + val + String.fromCharCode(0x2612);
    });
});
```

匹配关键字`a`以后，变成

```html
progr☑a☒mer
```

匹配不到`span`字符串，匹配结束。

最后再将这两个特殊符号替换为`'<span class="highlight">'`和`'</span>'`，在正则中使用`+`，可以匹配多个特殊字符，也就解决了用户输入输了两个以上相同的关键字并造成浏览器额外创建`span`元素的情况，当然你也可以再一开始就去重。
所以最终正确结果是

```js
string = string.replace(/\u2611+/g, '<span class="highlight">').replace(/\u2612+/g, '</span>');
```

```html
progr<span class="highlight">a</span>mer
```

以上答案正确与否，取决于 ☑ 和 ☒ 会不会出现在被替换文字中。

## 最终代码

```js
var keyword = input.value;
var string  = 'The string will be highlighted'；
// replace all preserved characters in regular express.
keyword = keyword.replace(/(?=[\\\{\}\[\]\(\)\*\+\?\^\$\-\=\!])/g, '\\');
// split keyword by space characters.
var keywords = keyword.split(/\s+/);
keywords.forEach(function(it){
  string = string.replace(it, function(val){
    return String.fromCharCode(0X2611) + val + String.fromCharCode(0X2612);
  });
});
string = string.replace(/\u2611+/g, '<span class="highlight">')
  .replace(/\u2612+/g, '</span>');
```

```css
.highlight {
    font-weight: bold;
}
```

## 总结

1. 高亮根据关键字搜索出来的记录不适合在客户端做，即时实现了，很可能是该高亮的没有高亮，不该高亮的高亮了。后端可以依靠巨大的知识库来高亮词性变换后的单词，甚至近义词。
1. 使用好强大的`String.prototype.replace`方法。
1. 正则中零宽断言在适当的时候，可以简化代码。
