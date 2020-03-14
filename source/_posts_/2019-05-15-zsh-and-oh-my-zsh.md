---
zsh 和 oh-my-zsh 使用方法
---

## `zsh`
在介绍[`zsh`](http://www.zsh.org/)之前，我们先说说`shell`。`shell`是一个用C语言编写的程序，它是用户使用`linux`的桥梁，通过它，用户可以访问操作系统内核的服务。和大多数`linux`系统一样，`macOS`系统默认是的`shell`也是`bash`。而今天我们要介绍的`zsh`也是一种类似的`shell`程序，兼容 `bash` 并提供了很多改进。

我们现在就打开终端应用来探索一下。
查看一下`macOS`系统中自带哪些`shell`程序
```bash
cat /etc/shells
```
你会看到以下结果
```bash
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```
说明`macOS`系统中自带6中`shell`程序。下面我们再看看系统目前使用哪种`shell`程序
```bash
echo $SHELL
```
如果你看到`/bin/bash`说明系统目前在用的是`bash`。我们接下来就把`shell`程序改成`zsh`
```bash
chsh -s /bin/zsh
```
重启一下终端应用，再执行`echo $SHELL`命令就可以看到`zsh`了。

配置`zsh`是一件麻烦的事，于是`oh-my-zsh`出现了。


## `oh-my-zsh`

[`oh-my-zsh`](https://ohmyz.sh/) 是一个的开源的，社区驱动的管理 `Zsh` 配置的框架。它捆绑了大量有用的函数，插件，主题和其他一些让你尖叫的东西...这是的官方的说辞。

### 安装
[官网](https://ohmyz.sh/)提供了两中安装方式，我们选择适合`macOS`的一种方法，如下：
```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

`zsh`的配置文件地址是`~/.zshrc`，用`vi`编辑器打开这个文件你就可以看到当前配置。

### 主题
在`~/.zshrc`中找到以下内容，我的在第4行。（备注：在`vi`中通过命令`:set nu`显示行号）
```bash
# Set name of the theme to load.
# Look in ~/.oh-my-zsh/themes/
# Optionally, if you set this to "random", it'll load a random theme each
# time that oh-my-zsh is loaded.
ZSH_THEME="robbyrussell"
```
可以看到当前的主题是`robbyrussell`，如果你对当前的主题并不满意，可以尝试修改它。根据`~/.zshrc`中注释说明，我们通过以下命令可以看到所有的预置主题：
```bash
ls ~/.oh-my-zsh/themes
```
也可以在[这个页面](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)根据截图找到喜欢的主题。然后修改`ZSH_THEME`的值即可。

### 插件
终于要说到`oh-my-zsh`最强大之处了——插件。在`~/.zshrc`中找到`plugins`（我的在第48行），就可以自定义启用的插件了，系统默认加载`git`。
```bash
# Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git)
```
正如注释里的说明和示例那样，在`plugins`后面的括号里可以添加其他插件，插件之间用空格分割。重启终端即可使用。

在[这个页面](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins)可以查看每个插件的说明。