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
Q: 页面空白，控制台出现警告“No layout: index.html”?
A: 这是由于没有下载指定主题`Anisina`。
```bash
cd themes
git clone https://github.com/Haojen/hexo-theme-Anisina.git Anisina
```