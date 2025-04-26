---
title: Windows开机启动软件、执行脚本，免登录账户
excerpt: 本文介绍了如何在Windows10中利用任务计划程序创建自启动任务，无需登录账户即可运行指定的软件或脚本。步骤包括打开任务计划程序，创建新文件夹，设置任务的常规信息，定义触发器和操作，以及添加运行条件。最后，通过重启电脑来检验任务是否成功执行。
date: 2025-04-26 15:30:00
index_img: http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456529418556.jpg
tags:
- Windows
- 开机启动
categories:
- Windows
---

# 前言
1. 电脑启动需要运行一些软件，但是不需要登录windows账户就能让软件运行起来，可参考本文章。
2. 测试系统Windows 10 专业版 22H2

# 一、打开任务计划程序
1. 我电脑上的是点搜索“任务计划程序”，可能每个电脑的搜索按钮不一样，自行查找
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527457393.jpg)

2. 打开后应该是长这样的
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527583996.jpg)

# 二、创建文件夹
1. 点击任务计划程序库、右键选择新建文件夹
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527678301.jpg)

2. 名字顺便，点击确定
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527760008.jpg)

3. 创建后如图、点击目录下应该是空的
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527828798.jpg)

# 三、创建计划任务
1. 在刚才创建的文件夹上右键，创建任务
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527895215.jpg)

2. 常规设置
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456527962430.jpg)

3. 触发器设置
**注意，此处设置为登录时**
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456528047402.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456528492076.jpg)

4. 操作设置（可选择程序或者脚本）
此处用浏览功能，找到`C:/Program files(x86)/fsmt/RebootHelper/RebootHelper.exe`
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456528557524.jpg)
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456529241762.jpg)

5. 条件设置
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456529351740.jpg)

# 四、创建完毕
1. 创建好后如图
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2025/04/26/17456529418556.jpg)

2. 重启电脑看是否执行，如未执行自行检查是否有配置错误

# 总结
本教程针对一些不需要登录windows账户就要执行的软件或者脚本。

{% note info %}
本文参考自以下博文，版权归原作者所有。

原文作者：null_17
原文地址：[windows开机启动软件、执行脚本，免登录账户](https://blog.csdn.net/qq_40622375/article/details/130352839)

{% endnote %}