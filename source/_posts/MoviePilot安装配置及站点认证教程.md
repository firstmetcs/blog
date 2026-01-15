---
title: MoviePilot安装配置及站点认证教程
excerpt: MoviePilot的作者听说也是nastools原作者，之前我们介绍过nastools《家庭影音系统及PT站玩家必备？nastools能解决什么痛点？》，MoviePilot的功能和使用上做了精简，更好上手。爱折腾的可以装一个，和nastools对比一下，说不定可以替代nastools，我就放弃了nastools用MoviePilot了。下面给大家简单说说MoviePilot的安装配置及站点认证。
date: 2026-01-14 13:20:55
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/moviepilot.png
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/moviepilot.png
tags:
- NAS
categories:
- NAS
---

MoviePilot的作者听说也是nastools原作者，之前我们介绍过nastools《[家庭影音系统及PT站玩家必备？nastools能解决什么痛点？](http://www.ptyqm.com/31566.html)》，MoviePilot的功能和使用上做了精简，更好上手。爱折腾的可以装一个，和nastools对比一下，说不定可以替代nastools，我就放弃了nastools用MoviePilot了。下面给大家简单说说MoviePilot的安装配置及站点认证。

如果你的NAS是群晖，那就可以直接大件安装MoviePilot，这就是群晖NAS相对其它极空间、绿联等NAS的一个好处用户群体大，套件生态比较完整。如果不是群晖NAS，那就只能docker来安装了，今天就先不讨论docker安装。

# 一、MoviePilot安装

首先是添加一下矿神的套件源，只支持DSM7.1或者以上的系统。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681056927.jpg)

然后安装一下**Python 3.11**

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681183728.jpg)

最后就可以安装MoviePilot了

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681335153.jpg)

# 二、认证站点

不进行认证是无法正常使用MP的，可以使用部分PT站进行认证。

认证站点支持：iyuu/hhclub/audiences/hddolby/zmpt/freefarm/hdfans/wintersakura/leaves/ptba /icc2022/xingtan/ptvicomo/agsvpt/hdkyl/qingwa/discfan/haidan/rousi

nastools，MoviePilot支持的认证站点都不支持mteam\hdsky\chdbits等大站，这是逼着大家收个小站，所以，平时大家看到有开放注册的小站，最好也收一个。

打开MoviePilot配置

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681602606.jpg)

然后加入如下图框框中的几行，下面我是以海胆这个PT站为例，如果你的是其它站点，可以参考官方的wiki：https://wiki.movie-pilot.org/zh/configuration

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681701121.jpg)

其中要注册的是这个ID不是你的用户名，是登录站点后，点击你的名字后，浏览器地址栏所显示的数字。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681806856.jpg)

# 三、添加站点

接下来打开MoviePilot登陆

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681955493.jpg)

用户名是admin，初始密码在配置那里可以查看

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683681701121.jpg)

登陆进去后**添加站点**

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682166150.jpg)

**COOKIE和User-Agent获取方式：**

以Edge浏览器为例，以在浏览器中右键，选择【检查】，然后打开并登陆你的PT站。在检查窗口选择【网络】标签，在列出的目录选择一个文件，直到能看到COOKIE和User-Agent。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682294056.jpg)

**RSS链接获取方式：**

点击PT站右上角获取RSS按钮

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682417354.jpg)

选择需要订阅的资源

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682517235.jpg)

点击生成RSS链接，我们这里用的是最后一个地址

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682628565.jpg)

# 四、添加QBittorrent下载器和目录

设置都很简单，填写一下下载器地址和账号密码就行了。把下载文件自动整理选上。后面还有一个媒体服务器设置，可以用emby、plex等等，根据需要设置，不用的可以不设置。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683682812858.jpg)

接下来设置下载保存目录和整理目录，这里要**特别注意的就是目录必须选择媒体类型**，建两个目录，媒体类型一个选择电影，一个选择电视剧。如果只建一个目录，媒体类型是全部，那就会添加不了下载，会提示找不到目录。

媒体库目录也对应建两个，整理方式建议选择硬链接。

以下是关于文件整理方式官方说明：

> **文件整理方式**
> 根据磁盘结构、空间大小、保种需要等综合决定使用哪种文件整理方式，在设定 -> 目录 -> 整理模式 中调整。
> 
> - **硬链接**：一份文件生成多个文件入口，但只占用一份存储空间，只有所有入口都删除后才能释放文件占用空间；可以修改硬链接后的文件名但不会影响原文件做种（不能修改文件内容）；要求在同一磁盘/存储空间/映射路径下才能硬链接。
> - **软链接**：类似于快捷方式，原文件删除后软链接即会失效；使用软链接时的原文件路径需要与生成软链接时的原文件路径保持一致，否则无法使用，也就是在docker环境下，映射前后的目录路径需要一致。
> - **复制**：复制一份副本，多占用一份空间。
> - **移动**：移动文件存储位置，会影响原文件做种。
> - **Rclone复制**：使用Rclone复制本地文件到网盘，需要自行映射rclone配置目录到容器中（/nt/.config/rclone）或在容器内使用rclone config完成rclone配置，网盘配置名称必须为：MP，可自行通过Docker添加环境变量传递参数优化传输，参考：https://rclone.org/docs/#environment-variables 。
> - **Rclone移动**：使用Rclone移动本地文件到网盘，其余与Rclone复制一致。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683683298468.jpg)

# 五、添加下载

点击浏览一个PT站的资源，点一下想下载的资源，然后确认即可自动添加到QB上进行下载。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683683454117.jpg)

下载完后还自动整理好，还刮削好了影片信息，非常方便高效。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683683543859.jpg)

如果你想整理后的文件夹名称**加上原文件名**，可以在MoviePilot配置里修改电影重命名格式。。我本人喜欢加上原文件名，这样可以知道这个**影片分辨率、影片格式、音轨格式、压制组**等信息。

```plaintext
MOVIE_RENAME_FORMAT={{title}}{% if year %} ({{year}}){% endif %}-{{original_name}}/{{title}}{% if year %} ({{year}}){% endif %}{% if part %}-{{part}}{% endif %}{% if videoFormat %} - {{videoFormat}}{% endif %}{{fileExt}}
```

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683683829723.jpg)

MoviePilot还有很丰富的插件可供你扩展功能使用，具体自己去研究了。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17683683938906.jpg)

关于MoviePilot的更详细介绍可以查看官方的WIKI：https://wiki.movie-pilot.org/

# 下载器设置（qb）

NASTool 的下载器支持 `qBittorrent/Transmission/115网盘/Aria2`，我这里使用qb，你也可以切换其他的。 直接到套件中心搜索并安装。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435809886902.jpg)

打开后自动跳转到 `http://ip:8085`, 默认账号密码 `admin / adminadmin`

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435810055103.jpg)

如果是BT用户，我们来修改一下下载文件夹就好了。个人建议的话，分个类，方便管理。我准备将下载的电影放到我的Download文件夹中，我创建了一个movie, 一个tv的分类，分别用户放电影和电视剧。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435810239464.jpg)

SSH 登陆，找到我们刚刚的目录，一般我们创建的存储池都在 `/volume`开头的文件夹下面，我们可以利用`ls /volume*` 列出所有的，然后找一下我们的目录在哪。如下图所示，我的目录在`/volume2/`下面。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435810397053.jpg)

这样我这里的下载地址就是 `/volume2/Download/movie` 和 `/volume2/Download/tv`。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435810775476.jpg)

默认路径修改一下。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435810898769.jpg)

然后添加两个分类。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435811015412.jpg)

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/14/17435811091830.jpg)

PT的用户需要`DHT`啥的关掉，我这里就不多说了，如果你是玩PT的你应该早就设置过。

然后我们到 `NASTool` 中 `设置 > 下载器` 中设置一下。

# 配置下载文件夹/媒体库

1、这里介绍一下我媒体库目录的结构，PTSSD为主目录，下面有两个文件夹一个是下载目录DATA，一个是硬链接目录DATA_LINK。通过MoviePilot实现的效果是资源下载到DATA目录，然后整理分类硬链接到DATA_LINK目录。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/15/17684501188751.jpg)

2、下载目录填写`/PTSSD/DATA`，其他保持默认，这样所有资源都会下载到DATA目录中，如果有分类需求，勾选下载目录自动分类，MoviePilot会自动在下载目录DATA中创建`电影`、`电视剧`等然后将对应资源存放到里面，`整理方式：硬链接`，`覆盖模式：按照大小覆盖`，`媒体目录填写硬链接目录`，其他保持默认，MoviePilot会自动创建资源分类目录。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/15/17684501638416.jpg)

# 下载器设置

如图下载器可以2选1，也可以都使用，输入对应下载器的地址、用户名、密码即可，http协议只需要输入ip和端口即可，https需要加上前缀，自动分类管理会在DATA目录下建立相对应资源类别的目录，因为后期我们会通过硬链接将资源整理到目录DATA_LINK中，所以在下载的时候不在做分类，设置完成之后点击保存按钮。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/15/17684501910094.jpg)


# 目录设置（重要）

1、首先是下载目录（资源下载的存放文件夹）MP的新版本目录设置对比老版本有了新的变化，支持了优先级并且可以设置多目录。

如下当我下载资源的时候MP会自动识别资源种类，如果是电影就放到第一个目录`/PTSSD/DATA/电影` 中，如果不是会依次向后匹配，直到匹配到正确的目录。

媒体类型中主类型主要用于资源类别分类，更方便管理，这里举例第一个目录我设置的是电影，媒体类别我设置成全部，并且勾选了下面的自动分类，MP会自动将下载资源识别，然后在电影目录中创建相对应类型文件夹存放下载资源，比如下载哥斯拉大战金刚，这个属于外语电影，会自动在电影目录下创建外语电影然后将资源存放进去。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/15/17684500802582.jpg)

2、媒体目录（资源被MP刮削后存放的目录）也叫硬链接目录原理同上对照设置即可。

![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2026/01/15/17684500929993.jpg)




{% note info %}
本文参考自以下博文，版权归原作者所有。

原文作者：Razeen
原文地址：[NAS折腾记(8)：群晖安装 NASTool 实现影音半自动化【多图】](https://razeen.me/posts/nas-10-nastool-install-and-basic-config/)

原文作者：PT邀请码
原文地址：[MoviePilot安装配置及站点认证教程，或可替代nastools](http://www.ptyqm.com/32620.html)

原文作者：折腾笔记
原文地址：[MoviePilot部署教程|新手指南|媒体整理|信息刮削|搜索下载|媒体订阅](https://blog.zwbcc.cn/archives/1711674204030)
{% endnote %}