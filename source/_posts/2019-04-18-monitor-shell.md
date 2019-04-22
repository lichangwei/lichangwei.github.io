---
title: 使用 Shell 实现服务的监控和重启
---

我们在公司内部通过[verdaccio](https://verdaccio.org/)创建了私有`npm`仓库，但是`verdaccio`不太稳定，每周都可能会挂掉一次两次的，在网上也没有找到好的解决方法。随着挂掉次数越来越多，痛定思痛，决定写个脚本主动监控`verdaccio`并在其挂掉以后自动重启。

## 判断服务是否正常运行

怎么判断`verdaccio`是否正常正常呢？如果是我肉眼判断的话，我直接打开`verdaccio`的主页，比如`http://example:4873`，如果能看到页面内容则说明服务正常，如果看到”无法访问次网站“等字样则说明服务挂掉了。为了实现自动化的监控和重启，我们必须通过脚本程序来做，在此我们可以通过[`curl`命令](http://man.linuxde.net/curl)来获取`http://example:4873`页面的响应码，如果返回`200`则说明服务正常，否则服务挂掉了。我们使用[`curl`命令](http://man.linuxde.net/curl)的参数`-I`只显示响应头。然后通过判断响应头中是否包含`HTTP/1.1 200 OK`字样来检查服务是否正常。因此就有了以下程序：

```bash
header=`curl -I http://example:4873`
if [[ $header =~ 'HTTP/1.1 200 OK' ]]; then
    echo 'ok'
else
    echo 'not ok'
fi
```

通过以上代码我们就可以监控服务是否正常了，我们再来拓展一下思路，其实我们还可以通过检查`verdaccio`服务进程是否存在来判断`verdaccio`服务正常与否。怎么判断进程是否存在呢？这里不再详细介绍，请参考一下代码：

```bash
# 查询有几个进程占用了`4873`端口
count=`lsof -ti :4873 | wc -l`
if [[ $count -ne 0 ]]; then
    echo 'ok'
else
    echo 'not ok'
fi
```

第二种方案不如第一种直观，我仍然把它写出来主要是提醒各位多多思考，不要拘泥于某一种方案。接下来我们仍然采用第一种更加直观的方案继续进行。

## 服务挂了以后自动重启

在上一节中我们已经能够检查服务是否正常了，在这一节中我们要实现的是自动重启服务，这个就非常简单了，想想我们是怎么启动`verdaccio`的，嗯，我们是在控制台中输入一下命令，`nohup verdaccio &`，其中`nohup`让命令永远执行下去，即使用户退出也没有关系，`&`让程序在后台运行。两者结合起来就可以程序永久地在后台运行。

{% codeblock lang:bash mark:5,6 %}
header=`curl -I http://example:4873`
if [[ $header =~ 'HTTP/1.1 200 OK' ]]; then
    echo 'ok';
else
    # 重启服务
    nohup verdaccio &
fi
{% endcodeblock %}

## 定期监控服务运行状况

通过前面两个步骤我们已经写出一个脚本，它会检查服务是否正常运行，如果服务挂掉了就会重启。这还是这个检查工作是一次性的，需要我们不停地执行脚本。当然可以通过[`crontab`命令](http://man.linuxde.net/crontab)（以后再介绍）实现，不过在这里我们将通过写一个死循环逻辑来实现每隔一段时间来检查服务是否正常运行。

{% codeblock lang:bash mark:1,2,10-12 %}
while true
do
    header=`curl -I http://example:4873`
    if [[ $header =~ 'HTTP/1.1 200 OK' ]]; then
        echo 'ok';
    else
        # 重启服务
        nohup verdaccio &
    fi
    # 每个10秒检查一次
    sleep 10s
done
{% endcodeblock %}

`sleep`命令会让程序暂停一段时间，很适合用在循环方式运行的监控脚本中，它有一个参数表示要暂停的时间，时间单位可以是`s`秒，`m`分钟，`h`小时和`d`天。默认为秒。我们也可以传入小数`0.1s`来实现毫秒级的睡眠，但是`sleep`命令只能保证`10ms`的睡眠，如果你对时间精度要求特别高的话，`sleep`命令就无能为力了。

## 重启服务以后记录日志

当服务挂了以后除了重启以外，还需要将重启行为记入到日志文件中，方便以后我们查看什么时候服务被重启了，最终代码如下：

{% codeblock lang:bash mark:3,4,14 %}
#!/bin/sh

# 获取脚本目录
shell_folder=$(cd `dirname $0`; pwd)

while true
do
    header=`curl -I http://example:4873`
    if [[ $header =~ 'HTTP/1.1 200 OK' ]]; then
        echo 'ok';
    else
        # 重启服务并记录日志
        nohup verdaccio &
        echo `date +%Y-%m-%d\ %H:%M:%S` "restart" >> $shell_folder/verdaccio.restart.log
    fi
    # 每个10秒检查一次
    sleep 10s
done
{% endcodeblock %}

执行命令`nohup sh verdaccio.sh &`就启动了一个守护进程，实现了每隔`10`秒检查一次服务是否运行正常，如果挂掉就会重启的功能，并且还会记录到日志中。

如果你是在`Ubuntu`服务器上运行该命令，可能会遇到这个错误`[[: not found`，这是因为`sh`只是一个符号链接，最终指向是一个叫做`dash`的程序，自`Ubuntu 6.10`以后，系统的默认`shell /bin/sh`被改成了`dash`。`dash(the Debian Almquist shell)` 是一个比`bash`小很多但仍兼容`POSIX`标准的`shell`，它占用的磁盘空间更少，执行`shell`脚本比`bash`更快，依赖的库文件更少，当然，在功能上无法与`bash`相比。所以在`Ubuntu`上我们需要指定使用`bash`，即`nohup bash verdaccio.sh &`。

## 如何取消服务自动重启
如果有一天，你想关闭`verdaccio`服务，守护进程就会检测到该服务挂掉，并自动重启该服务。导致你想关闭该服务也不行了。所以我们首先要先关闭守护进程，如何关闭呢？我们可以使用[`jobs`命令](http://man.linuxde.net/jobs)查看守护进程ID，然后杀掉该进程。
```bash
jobs -l
kill -9 <id>
```

## 针对`verdaccio`的特殊方案
其实`verdaccio`是用Node写的，因此可以通过`pm2`让其达到自动重启的功能，命令如下：
```bash
# 启动
pm2 start which verdaccio 
# 停止
pm2 stop which verdaccio 
# 查看日志
pm2 show verdaccio
```
