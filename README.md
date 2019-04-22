博客
===

## 启动
```bash
# 安装命令行工具
npm install hexo-cli -g
# 安装依赖
npm i
# 启动服务
hexo server
# 部署
hexo deploy --generate
```

## 问题
### 1. 页面空白，控制台出现警告“No layout: index.html”?
这是由于没有下载指定主题`next`。
```bash
cd themes
git clone git@github.com:theme-next/hexo-theme-next.git next
```

### 2. 如果高亮代码行？
使用以下方式编写代码，并通过mark属性指定要高亮的代码行
```md
{% codeblock lang:bash mark:3,4,14 %}
    code hear
{% endcodeblock %}
```