---
title: iStoreOS使用阿里云ddns+https
excerpt: 本文简单介绍使用iStoreOS配置阿里云ddns和https
date: 2024-01-30 20:50:20
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/istoreos.png
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/istoreos.png
tags:
- 软路由
- ddns
- aliddns
categories:
- 软路由
---
{% note info %}
本文由 Fluid 用户授权转载，版权归原作者所有。

本文作者：LSYZZs
原文地址：https://blog.csdn.net/lesiyu116/article/details/127229081
{% endnote %}

**首先声明这篇文章不是保姆级的。假定看这篇帖子的伙伴是有一定基础的。**
* 在阿里云上购买一个域名 .cn 价位大概在 40 rmb 一年
* 然后你要为这个域名备案。国内的没办法。（如果不想这么麻烦可以找其他域名备案商）
* 申请一个 AccessKey 子账号
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190233252.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190287101.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190340413.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190405626.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190546650.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190636303.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190693529.jpg)
AliyunDNSFullAccess

点击确定。阿里云上的操作就告一段落了
下面是iStoreOS

在服务中找到 动态DNS 选项卡 点开
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066190915554.jpg)
点击添加新服务
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066191036007.jpg)
点击创建服务
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066191146820.jpg)
到了这个页面 让我们切换到阿里云上。
在阿里云的域名解析服务中 新建一条 A 记录 。 记录值随便填![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066191286145.jpg)
创建好后回到 iStoreOS
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066191413083.jpg)
保存并引用。
接着设置端口转发 将 80 端口 和 443 端口转发出去。这一步就不多讲了。有需要的可以百度。很多的大神都有讲过。

**在 iStoreOS 的软件商城中下载 uhttpd**
**下面就是配置 https 和证书了。**
这就不得不说为什么要在阿里云上注册域名了。因为在阿里云上注册域名有免费的ssl证书可以用。每人每年有18个免费证书可以用。
生成证书可以安装阿里云的官方教程生成。生成后下载 nginx的证书
下载后的证书不能直接用。因为阿里云的 nginx的证书是 pem的
而uhttpd 中使用的证书是 crt 的 所以需要转换一下
`openssl x509 -in 证书路径/证书.pem -out 证书导出路径/证书.crt`
将 .key .crt 的证书上传到 iStoreOS 服务器中
在 uhttpd 中 将证书地址切换到 上传的证书路径上
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/30/17066192201679.jpg)
设置后点击保存斌应用。

**ok到这里https的证书就设置完成了。剩下的就是在浏览器中输入 https:你的域名:443转发的那个端口号了。**
例如 443端口转发到 18443 那么就是 https:域名:18443
