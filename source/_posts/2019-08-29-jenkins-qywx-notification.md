---
title: 在 Jenkins 中添加企业微信通知
---

## 在Jenkins中安装插件`Qy Wechat Notification Plugin`
进入插件管理页面，有两种方法：
1. 一步一步点击进入：Jenkins首页->系统管理->插件管理→切换到“available”Tab标签，
2. 通过修改地址栏`http://<domain>/pluginManager/available`直接进入该页面
在页面右上角Filter输入框中输入插件名称`Qy Wechat Notification Plugin`，并在搜索结果中找到该插件并安装。

## 在企业微信群中添加机器人
选择你要通知的企业微信群，右键选择”添加群机器人“，然后按照向导程序一步一步地操作，在这期间只需要你给机器人起个名字。由此你已经成功的创建了群机器人，在群对话窗口右下方，你可以看到刚刚创建的机器人。
![](../../../../images/jenkins-qywx/qywx-robot.png)

右键查看机器人资料，你就可以看到该机器人的`Webhook 地址`，拷贝下来接下来会用到。

## 给Jenkins任务添加企业微信通知
选择你的Jenkins任务，并进入配置页面，滚动到页面底部，点击”Add post-build action“按钮，选择”企业微信通知配置“，然后可以看到如下界面。
![](../../../../images/jenkins-qywx/add-post-build-action.png)

在`Webhook 地址`中添加第二步中创建的机器人的 Webhook 地址，点击保存即可。剩下的三个字段用来配置需要 `@` 的关键人，比如构建失败了通知项目负责人。


至此配置完成，在以后该任务执行前后，你都可以收到群机器人发送的通知消息，点击消息中的”查看控制台“可以直接查看该任务的执行情况，是不是很方便呢？
![](../../../../images/jenkins-qywx/robot-message.png)
