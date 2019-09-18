---
title: Ubuntu18.04美化
date: 2019-09-16 16:18:59
tags:
	- ubuntu
categories:
	- Linux
---

## 前言

ubuntu桌面比较简陋，这就驱使很多人想DIY自己的一套桌面主题，如果是从零开始配置和美化ubuntu18.04，可以建议先参考[Ubuntu 18.04配置及美化](https://blog.csdn.net/ice__snow/article/details/80152068)

## 工具

> 工欲善其事，必先利其器

要对ubuntu进行美化就必定要用到神器gnome-tweak-tool，这是 Gnome 官方发布的一款 Gnome 调节软件, 借助这款软件, 我们可以更好地管理主题, 扩展, 字体 以及系统行为等设置项。安装方式很简单，命令行下执行

```shell
sudo apt install gnome-tweak-tool
```

安装tweak好之后打开界面如下

<div align="center">
    <img src="/images/tweak.png">
</div>

## gnome扩展

[Gnome Shell Extensions](https://extensions.gnome.org/) 是 Gnome 的一系列插件, 类似 Chrome 的插件, 可以起到系统增强的作用。借助chrome的插件可以方便的访问gnome扩展网站并实现一键添加或删除gnome扩展程序，具体安装过程两步

* 安装 Chrome 扩展程序 [GNOME Shell integration](https://chrome.google.com/webstore/detail/gnome-shell-integration/gphhapmejobijbbhgpjhcjognlahblep)
* 安装 主机连接器 `sudo apt install chrome-gnome-shell`

接下来我们就可以在网站 [GNOME Shell Extensions](https://extensions.gnome.org/) 安装 gnome 扩展了。

通过搜索找到自己心仪的扩展程序，点击进入详情页面，切换详情页面的“OFF”按钮即可安装对应扩展，如下图红圈标示

<div align="center">
    <img src="/images/gnome-shell-extension.png">
</div>

点击红叉即可卸载该扩展程序。有了tweak和gnome shell extension之后就可以开始DIY自己的ubuntu桌面了。



## 安装gnome扩展

先安装一些好用的扩展程序以帮助我们提高工作效率，可参考[简书：Ubuntu 18.10 美化](https://www.jianshu.com/p/5bd14cbf7186)

* dash to dock 优化 Ubuntu 默认的 dock
* User Themes 自定义 shell 主题
* Coverflow Alt-Tab 优化 Ubuntu 默认窗口切换动作
* Gnome Global Application Menu 将当前程序的菜单项提取到状态栏
* NetSpeed 显示网速插件
* Clipboard Indicator 提供剪切板历史记录功能
* Drop Down Terminal 可以从屏幕上快速弹出一个终端
* Recent Items 快速打开最近打开过的文件
* Places Status Indicator 利用下拉菜单快速打开驱动器上的常用位置
* Dynamic Top Bar 动态调整状态栏透明度
* Hide top bar 隐藏顶栏, 可以设置为鼠标靠近屏幕上边沿时显示顶栏
* Top Panel Workspace Scroll快速切换工作区
* Gravatar 把你的 Ubuntu 用户头像设置成你的 Gravatar 头像.
* TopIcons Plus 将传统托盘图标移动到顶部面板 (Wine 程序救星)

按下 Alt + F2,输入 r，回车重启 gnome。



## 安装theme和icon

有了便捷的扩展程序以后，再搭配一个让人赏心悦目的主题岂不美哉。maxOS的桌面风格很受程序猿的喜欢，所以先安利一款macOS主题桌面[MCHigh Sierra](https://www.gnome-look.org/p/1013714/)，具体的制作过程可参考[Ubuntu18.04主题更换为Mac OS high Sierra美化教程](https://ywnz.com/linuxmh/2105.html)，按照里面的教程一步一步来即可制作出属于自己的macOS主题桌面。更多的MacOS主题安装教程可以参考[给Ubuntu18.04(18.10)安装mac os主题](https://www.cnblogs.com/feipeng8848/p/8970556.html)

> <font color='red'>Caution:</font>
>
> Sierra的原始资源地址[McHigh Sierra](https://www.gnome-look.org/p/1013714/)仅提供了gnome应用主题，没有提供图标风格icon，想要获取对应主题的icon可从[Ubuntu18.04-tutorials-themes](https://github.com/vbay/CSDN-CODE/tree/master/Ubuntu18.04-tutorials-themes)获取

除了macOS主题之外还有很多其它好看的主题，可以根据个人喜好在[gnome-look](https://www.gnome-look.org/)中进行查找。比如我个人喜欢的一套主题是[Vimx-beryl Theme](https://github.com/vinceliuice/vimix-gtk-themes)及其配套icon[Vimx-beryl Icon](https://github.com/vinceliuice/vimix-icon-theme)，这套主题的具体安装教程可参考[知乎：Ubuntu 18.04 LTS 安装、美化](https://zhuanlan.zhihu.com/p/37314255)

> <font color='red'>Caution:</font>
>
> 下载的theme可以放在系统themes目录/usr/share/themes下，也可以放在用户目录\~/.themes下。而下载的icon可以放在系统icons目录/usr/share/icons下，也可以放在用户目录\~/.icons下



## 安装gnome shell

[Flat Remix](https://www.gnome-look.org/p/1013030/)个人挺喜欢的一款gnome shell风格，可以通过添加源在命令行下安装

```shell
sudo add-apt-repository ppa:daniruiz/flat-remix
sudo apt-get install flat-remix
sudo apt-get install flat-remix-gnome
sudo apt install gnome-shell-extensions
```

然后重新打开tweak，在扩展一栏中将User themes打开之后即可切换gnome shell的风格了



## 安装zsh

zsh是mac默认的shell，而ubuntu的默认shell是bash。相比bash，zsh配合[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)拥有更丰富的主题，使得命令行更为美观。

```shell
sudo apt install zsh
# 切换到zsh
chsh -s /bin/zsh 
```

重新登录shell即可转换到zsh，接下来在用户主目录下安装`oh-my-zsh`

```shell
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

接着安装插件`highlight`，高亮语法

```shell
 cd ~/.oh-my-zsh/custom/plugins
 git clone git://github.com/zsh-users/zsh-syntax-highlighting.git
```

在`Oh-my-zsh`的配置文件中`~/.zshrc`中添加插件

```shell
# 括号中的插件名以空格分隔
plugins=( [plugins...] zsh-syntax-highlighting)
```

设置zsh主题

```shell
# 个人觉得比较好看的zsh主题有: robbyrussell(default), agnoster, bira
ZSH_THEME="robbyrussell" 
```

> <font color='red'>Caution:</font>
>
> 其中[agnoster](https://github.com/agnoster/agnoster-zsh-theme)主题比较像之前在bash下使用的powerline，由于包含特殊的字体符号，需要安装额外的字体 [Powerline-patched font](https://github.com/Lokaltog/powerline-fonts)才能支持主题正常显示，ubuntu下可直接使用apt安装
>
> ```shell
> sudo apt-get install fonts-powerline
> fc-cache -vf /usr/share/fonts/  #更新系统的字体缓存
> ```

zsh主题定制可以参考[oh-my-zsh终端用户名设置（PS1）](https://www.jianshu.com/p/bf488bf22cba)，zsh的一些主题例如agnoster会自动添加命令行头名user@host，如果觉得这种形式使得命令行看起来很臃肿，可以在\~/.zshrc中可以设置DEFAULT_USER来避免

```shell
DEFAULT_USER=brooksj
```

避免命令行头名臃肿还可以通过prompt_context() {}来设置

```shell
# 隐藏用户名和主机名
prompt_context() {}                                                           

# 只保留用户名，隐藏主机名
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi  
}
```

个人倾向于只保留用户名，最后使配置生效

```shell
source ~/.zshrc
```



## 安装background

backgound的图片可从壁纸网站[wallhaven]([https://wallhaven.cc](https://wallhaven.cc/))下载，然后放到/usr/share/background中，右击桌面切换背景即可



## 个人配置展示

个人所使用的应用程序、光标、图标、shell在tweak中配置如下图所示

<div align="center">
    <img src="/images/tweakconfig.png">
</div>

个人用到的扩展(gnome shell extensions)为

1. Clipboard Indicator 提供剪切板历史记录功能
2. dash to dock 优化 Ubuntu 默认的 dock
3. User Themes 自定义 shell 主题
4. Coverflow Alt-Tab 优化 Ubuntu 默认窗口切换动作
5. Drop Down Terminal 可以从屏幕上快速弹出一个终端
6. Gravatar 把你的 Ubuntu 用户头像设置成你的 Gravatar 账户头像.
7. NetSpeed 显示网速插件
8. Recent Items 快速打开最近打开过的文件
9. Places Status Indicator 利用下拉菜单快速打开驱动器上的常用位置
10. Top Panel Workspace Scroll快速切换工作区



<font size=4 color='green'>参考链接：</font>

1. [Ubuntu 18.04配置及美化](https://blog.csdn.net/ice__snow/article/details/80152068)---从零开始配置的建议入手
2. [Ubuntu 18.04 LTS 安装、美化](https://zhuanlan.zhihu.com/p/37314255)
3. [Ubuntu18.04主题更换为Mac OS high Sierra美化教程](https://ywnz.com/linuxmh/2105.html)
4.  [给Ubuntu18.04安装mac os主题](https://www.cnblogs.com/feipeng8848/p/8970556.html) 
5. [Ubuntu 18.10 美化](https://www.jianshu.com/p/5bd14cbf7186)









