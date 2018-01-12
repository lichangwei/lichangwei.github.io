---
title: IIS的URL重写（反向代理）
---

以前我使用过`Apache`或`Nginx`实现反向代理，但是目前后端使用的是`.Net`语言，服务器操作系统是`Windows Server`，并且后端应用程序就部署在`IIS`上，因此不打算再安装`Nginx`，改为使用`IIS`。

在操作过程中，遇到各种问题也耽误了不少时间，现在将一些操作流程记录下来，以备再用。

## 项目简介
项目采用前后端分离方式开发，两个代码仓库，并部署为两个应用。后端接口地址全部以`/api`打头，方便简化转发规则。现在就需要在前端应用中将`/api`打头的请求转至后端应用。

## IIS默认不支持反向代理，需要额外安装插件
很难想象`IIS`(Internet Infomation Service)竟然不默认支持反向代理，需要安装官方提供的两个插件。
安装插件：
1. [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing)
2. [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
安装完成之后重新启动一下`IIS`.

## 启用反向代理功能
1. 双击`IIS`根目录，双击`Application Request Routing Cache`，双击右侧的`Server Proxy Settings`。
![](../../../../images/iis-url-rewrite/1.click-iis-root-select-arr.png)
![](../../../../images/iis-url-rewrite/2.server-proxy-setting.png)
2. 勾选`Enable proxy`
![](../../../../images/iis-url-rewrite/3.enable-proxy.png)

## 添加入站规则
1. 点击前端应用，双击`URL Rewrite` -> `Add Rules`(新建规则) -> `Blank rule`(空白规则) 
![](../../../../images/iis-url-rewrite/4.select-url-rewrite.png)
2. 填写入站规则信息
![](../../../../images/iis-url-rewrite/5.add-rule.png)

3. 填写转发条件
![](../../../../images/iis-url-rewrite/6.add-condition.png)

4. 填写操作部分
![](../../../../images/iis-url-rewrite/7.add-operation.png)

5. 点击右侧应用

## 最后
反向代理的配置相比`Nginx`等复杂太多了，还需要额外理解IIS自己定义的一些概念，比如入站规则等。相信大部分刚毕业的同学跟我一样，一度特别喜欢有操作界面的软件，随着对系统掌握程度的增加，这个想法会慢慢的逆转。

