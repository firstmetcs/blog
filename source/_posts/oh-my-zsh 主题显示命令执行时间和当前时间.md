---
title: oh-my-zsh 主题显示命令执行时间和当前时间
excerpt: 本文介绍如何在oh-my-zsh主题中显示命令执行时间。通过修改默认主题robbyrussell，添加特定函数preexec和precmd，可以实现在提示符中显示上一条命令的执行时间。此外，提供了一键安装脚本，方便用户快速配置。
date: 2026-04-07 14:07:40
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/oh-my-zsh.webp
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/oh-my-zsh.webp
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

本文作者：funnyang
原文地址：https://xiaoyuyu.cn/post/oh-my-zsh-display-time.html
{% endnote %}

参考《[oh-my-zsh主题添加命令显示执行时间和当前时间](https://blog.csdn.net/weixin_41100576/article/details/106334391)》进行了修改，兼容 vscode 终端

以 robbyrussell 为例

```bash
cd .oh-my-zsh/themes
vim robbyrussell.zsh-theme
```

添加如下内容：

```bash
function preexec() {
  timer=${timer:-$SECONDS}
}

function precmd() {
  if [ $timer ]; then
    timer_show=$(($SECONDS - $timer))
    if [[ $timer_show -ge $min_show_time ]]; then
      RPROMPT='%{$fg_bold[red]%}(${timer_show}s)%f%{$fg_bold[white]%}[%*]%f'
    else
      RPROMPT='%{$fg_bold[white]%}[%*]%f'
    fi
    unset timer
  fi
}

autoload -Uz add-zsh-hook
add-zsh-hook preexec preexec
add-zsh-hook precmd precmd
```

效果：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/04/07/17755421551976.jpg)
我实际使用的：

```bash

function preexec() {
  timer=${timer:-$SECONDS}
}

function precmd() {
  if [ $timer ]; then
    timer_show=$(($SECONDS - $timer))
    RPROMPT='%F{red}(${timer_show}s)%f[%*]%f%{$reset_color%}'
    unset timer
  fi
}

autoload -Uz add-zsh-hook
add-zsh-hook preexec preexec
add-zsh-hook precmd precmd
```
