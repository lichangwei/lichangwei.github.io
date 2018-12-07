---
title: 设计规范
---

一. 命名规则 !important

<div id="list-inbox" class="mail-list">list information</div>  //正确
<div id="listInbox" class="mailList">list information</div>  //错误
id 与 class 均采用中划线命名法，不得驼峰命名法
二. 合理命名
<div id="sidebar">side information</div>  //推荐
<div id="col-left">side information</div>  //不推荐
根据内容而不是表象给 id 和 class 合理命名
三. 标签语义化
<em>这里是强调文本</em>  //用em标签标识强调文本
<strong>这里是重点强调文本</strong>  //用strong标签标识重点强调文本
根据内容选择合适的html标签，尽量避免写入无意义标签，尝试使用微格式(MicroFormats)
人可以通过视觉的划分判断内容的语义,搜索引擎看到的只是代码。搜索引擎只能通过标签来判断内容的语义。页面的很大一部分流量是来自搜索引擎的，所以我们要使页面尽可能地对搜索引擎友好
h标签的语意是标题，搜索引擎对这个标签比较敏感，特别是h1,和h2。一个语义良好的页面，h标签应该是完整有序没有断层的。也就是说，要h1,h2,h3,h4这样推下来，不要h1,h3,h4，漏掉h2。一个结构良好的网页，h标签可以组织起一个网页的大纲
四. 良好的文档结构
清晰合理的文档结构，对SEO更友好，方便行为层操控DOM结构，使之易于维护、扩展和阅读
五. 分离的思想
<div style="padding:10px">sidebar information </div>  //直接插入样式，不推荐
<div onclick="showMenu()">sidebar information </div>  //直接注册事件，不推荐
结构层、表现层和行为层的分离，尽可能的不要在 HTML 结构标签中直接插入样式或者 JavaScript 脚本代码

书写规范
一. 标签的元素和属性名都必须小写 !important
备注：

1. IE 另存为的 html 文件，会把标签全改为大写，这是不符合标准的，不能直接复制粘贴复用。如有这方面的需要，请用 Firefox 保存，或者用工具(比如:dreamwaver)将标签全部转为小写。
2. 另一个值得注意的是，通过内联方式(<a href="#" onmouseover="" >link</a>)注册的事件，事件名必须全部小写。
   <INPUT NAME="inputX" VALUE="" onMouseOver="fn()" /> //错误
   <input name="inputX" value="" onmouseover="fn()" /> //正确
   二. 标签必须关闭 !important
   备注：自封闭的标签必须自封闭，如：<br />, <input />等，所有自封闭的 HTML 标签请点击 这里
   备注：标签未关闭的错误常见于模版输出和挖天窗的过程中，请相关人员要特别注意分析好页面结构
   <input name="inputX" value=""> //错误
   <input name="inputX" value=""></input> //错误
   <input name="inputX" value="" /> //正确 (最后的斜杠“/”与前面的字符串之间要有一个空格)
   三. 标签必须正确嵌套 !important
   <div><p>text</div></p> //错误
   <div><p>text</p></div> //正确
   扩展阅读: HTML 标签嵌套规则
   四. 标签属性必须使用双引号 !important
   <input name='a' id=b value= /> //错误
   <input name="a" id="b" value="" /> //正确
   五. 属性值不能简写，必须是名值对的形式(name=”value”) !important
   selected="selected" //类似的还有 disabled , checked , readonly , noresize
   六. 自定义属性，必须以 data-开头(data-name=”value”)，多个单词用‘-’连接 !important
   <li name="a" value="b" fid="1234" country-code="cn">context</li> //错误
   <li data-name="a" data-value="b" data-fid="1234" data-country-code="cn">context</li> //正确
   使用 data-开头的自定义属性，是 HTML5 新增加的功能，可以使用更多 HTML5 的新特性
   HTML5 JavaScript API 提供了访问这些自定义属性的方法（除了 setAttribute/getAttribute 以外）DOM.dataset 点击查看
   七. 使用自定义属性(data-role="xxx")代替 class 做“js 钩子” !important
   <li class="J-item">context</li> //不推荐
   <li data-role="item">context</li> //推荐使用
   使用 data-role="xxx"来代替原来的 class="J-xxx"可以更好地将 css 与 js 分离，利于发展维护
   八. 属性值特别是依赖后端输出的属性值必须经过 html encode，以防止 XSS 攻击 !important
   其中 a 标签 href 属性 PATH 部分必须经过 encode.
   前端操作 Cookie 时，涉及用户输入数据的部分也要特别注意这一点，比如: 网站中的 Recent Search 记录
   安全输出 velocity 变量的宏请猛击这里
   九. 标签内容，服务器端输出的，根据情况确定是否需要经过 encode !important
   纯文本，必须。比如:链接文本
   包含 html 输出时不能编码的，要有意识地联系工程师做好后端输入过滤。比如：富媒体编辑器内容
   安全输出 velocity 变量的宏请猛击这里
   十. 特殊字符尽量用相应的符号实体代替
   特殊字符 对应的符号实体
   & &amp;
   < &lt; > &gt;
   空格 &nbsp;
   查看更多？猛击这里下载整理的电子书
   十一. 关于 table 标签
   table 标签中，如果使用 thead、tfoot 以及 tbody 元素之一，就必须使用全部的元素，它们的出现次序是：thead、tfoot、tbody 。
   扩展阅读: 标准化 table 标签
   十二. 代码缩进
   在书写代码的时候, 缩进并不会影响页面的最终表现, 但使用适当的缩进能使代码更具可读性, 我们推荐的缩进方法是当你开始一个新的元素时缩进一个 Tab 位(按一次 Tab 键——4 个空格的长度). 另外, 记得, 关闭元素的标签与开始标签对齐.示例：
   <div class="container">
   hello, Alibaba!
   </div>

注释规范
如非必要，HTML 代码中不允许出现 HTML 注释（系统自动生成的除外），用 velocity 的注释代替。
