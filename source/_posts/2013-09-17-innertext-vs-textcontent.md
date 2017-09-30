---
title: innerText vs textContent
---

为什么innerText会导致重排？我在我的上篇[关于重排的文章](http://kellegous.com/j/2013/01/26/layout-performance)末尾提出了这个问题。这里简单地总结一下，有些DOM API可能会因为不必要的重排，从而导致严重的性能问题。我将在这些文章中，着重提出这些年我发现的关于Web性能的古怪问题。  

As with many other things in browsers, innerText’s behavior seemed to have happened due to overlapping (and, perhaps, under-defined) use cases. 当你要获取DOM节点中的文本时，你会遇到两个问题，节点中原始文本是什么？用户看到的文本是什么？虽然类似，但是显然有所不同。在浏览器中，可以通过textContent获得前者，通过innerHTML获得后者。  

我们用以下例子来解释他们的最重要的不同点，看看以下HTML节点的innerText和textContent。  
```html
<div id="t"><div>lions,
tigers</div><div style="visibility:hidden">and bears</div></div>
```
innerText: `lions, tigers`  
textContent: `lions,\ntigersand bears`  

注意以下不同点：（1）.那些没有显示出来元素也不会出现在innerText中（2）innerText中的换行符遵循布局中换行符的规则。理解innerText最好的方式就是，它大概就是你选择该节点的内容然后拷贝获得的文本。而textContent是各个子节点的文本内容组成的文本。  

## innerText可能不是你想要的
innerText最大的特点是需要从布局系统中获得一些信息然后决定文本是怎样显示给用户的，这就是为什么innerText成为一个让性能变糟的一个属性。很多库喜欢用`innerText`而不用`textContent`仅仅是因为IE中先有`innerText`。下面让我们演示选择`innerText`和`textContent`时对性能的影响。  

在基于Webkit的浏览器中，你可以看到很大的性能差异，大概`300ms`对`1ms`。在IE9中，你可以看到较好的性能以及较小的差异，这是因为

## innerText可能不是你想要的另一个原因
虽然innerText被广泛使用，但它不是规范的属性。它能够存活下来是因为IE时代的广泛使用。
On a WebKit browser, you should see a significant performance difference (~300ms vs ~1ms). On IE9, you’ll see better performance and a much smaller difference. It is clear that IE avoids computing a full layout and probably uses a special code path that computes only what is needed for innerText (which really isn’t much). If you are using Firefox or Opera, you may be scatching your head. Keep reading.

While one could certainly conceive of use cases for innerText, most callers just assume that innerText and textContent are identical. You will see the expression node.innerText || node.textContent still being used in a number of libraries. Unfortunately, that leaves the door open for some unexpected performance problems. It is much wiser to prefer textContent these days.

Another reason innerText is probably not what you want
While it is still widely used, innerText is not standard. It is a bit of behavior that has lived on due to wide use during the Internet Explorer era. It’s heavy use back then is probably the reason IE seems to have a specialized code path. To this day, it is not present in Firefox (wise decision on their part) and its behavior still varies widely in the browsers that do support it. Opera, for instance, merely computes textContent when you try to access innerText. This is why it outperforms WebKit in the example I show. When I use the expression “browser landmine”, innerText is what I have in mind. To quote my good friend, Joel Webber, “it’s slower, but at least it doesn’t work as you would expect.”

