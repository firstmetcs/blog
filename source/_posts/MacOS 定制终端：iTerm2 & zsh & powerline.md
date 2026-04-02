---
title: MacOS 定制终端：iTerm2 + zsh + powerline
excerpt: 本文主要记录了 MacOS 终端使用iTerm2配置zsh的定制。
date: 2026-04-02 12:08:08
index_img: http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751036009449.jpg
banner_img: http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751036009449.jpg
category_bar: true
tags:
- iTerm2
- zsh
- 终端
categories:
- iTerm2
---
{% note info %}
本文由 Fluid 用户授权转载，版权归原作者所有。

本文作者：躺在云上数星星
原文地址：https://www.jianshu.com/p/3c1ae9ff6047
{% endnote %}

# 写在前面

最终效果图

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751036009449.jpg)

想配置实用的终端，请看我另一篇文章：[MacOS 终端工具、插件推荐](https://www.jianshu.com/p/c9040b4321ae)

# 安装iTerm2

可以直接去官网下载：
[https://www.iterm2.com/](https://www.iterm2.com/)

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751041061251.jpg)

初始的样子可能比价丑，但没关系，我们一步步优化
可以先设置一下颜色主题，如下图

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751041319041.jpg)

我使用的是吸血鬼主题，非常出名，具体下载方式，去官网：
[https://draculatheme.com/](https://draculatheme.com/)
下载完，导入进来就行，选中就行了。

# 安装powerline

通常可以使用pip来安装，没有pip的先安装个python,

```bash
pip install powerline-status
```

然后安装对应的字体
链接：
[https://github.com/powerline/fonts/blob/master/Meslo%20Slashed/Meslo%20LG%20M%20Regular%20for%20Powerline.ttf](https://github.com/powerline/fonts/blob/master/Meslo%20Slashed/Meslo%20LG%20M%20Regular%20for%20Powerline.ttf)

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751041724886.jpg)

下载字体后，直接安装就行

# 安装oh my zsh

安装方法有两种，可以使用curl或wget，看自己环境或喜好(当然，curl或者wget都可以通过homebrew来安装，如果没有的话)：

```bash
# curl 安装方式
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

```bash
# wget 安装方式
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

安装完成之后可以修改下shell，如果你默认不是zsh的话

- 设置默认的shell为Oh-My-ZSH
```bash
chsh -s /bin/zsh
```

安装完成之后用户目录下（~）默认会有一个`.zshrc`文件，如下图

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751042219282.jpg)

这个是用来配置zsh的
用vim或者 文本编辑器打开
可以先设置一下主题：有非常多的主题可以选择，如下图，

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751042525440.jpg)

但是为了适应powerline我们可以选择：`agnoster`
如下图：

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751042682272.jpg)


当然，我这个自己定制的，只需要复制另外一份，修改下，就行

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751042812926.jpg)

具体修改的地方如下(可以避免名字过长)：
```bash
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751042985947.jpg)

配置iTerm2
然后需要配置下iTerm2
如下：

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751053563845.jpg)

这样应该就大功告成了。

最后提供下agnoster-custom.zsh-theme的主题地址，我自己配置的：[https://github.com/qykingle/myconfig/blob/master/agnoster-custom.zsh-theme](https://github.com/qykingle/myconfig/blob/master/agnoster-custom.zsh-theme)

然后关于底部状态栏的设置：

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751053738100.jpg)

还有评论中提到的蓝色小箭头中的设置方式：

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/02/17751053872432.jpg)
