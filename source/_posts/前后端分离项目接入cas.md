---
title: 前后端分离项目接入cas
excerpt: 本文简单介绍基于Spring Security OAuth2和umi request的前后端分离项目接入CAS。
date: 2024-04-09 12:10:00
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/cas.jpeg
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/spring-security-oauth.jpeg
tags:
- Spring Security OAuth2
- CAS
- 前后端分离
- umi
categories:
- Spring Security OAuth2
- CAS
---
# 项目简介
后端采用spring security oauth2
前端采用antd + umi request
cas使用Apereo/cas 5.3

# 项目现状
前后端完全分离，采用无状态的restful接口方式，身份和权限校验使用oauth redis token（grant_type为password）

# 实现思路：
1. 前后端分离框架系统通过cas认证后状态和信息由内部管理，无状态的接口请求使用redis token进行状态管理（jwt亦可，下同）
2. 系统不由页面或者请求发起redirect重定向，而是由拦截器拦截到响应码进行跳转，避免需要多处实现重复逻辑
3. 由于是前后端分离，为保证登陆后跳转到原页面的用户体验，不由后端发起跳转，而是由前端发起跳转，登陆成功跳转后回到原页面

# 主要处理流程
1. 由后端接入cas验证，为oauth添加新的grant_type为cas_ticket（原来使用password）参照：[Spring Security OAuth2+CAS（添加cas模式）](https://blog.csdn.net/a294634473/article/details/109692209)
2. 前端访问后端资源，若未登录返回401 status code，修改原跳转登录页的逻辑为跳转到sso地址并携带当前前端页面地址（https://sso-server/cas/login?service=http://front-end/current-path?param=test)
3. cas认证成功后跳转到原前端页面地址并携带参数：http://front-end/current-path?param=test&ticket=ST-32-pN6gmwToJ3equoYzdkXrr4vOeVoiZ2ze31var6xtl7knk0qv0Z
4. 前端初始化时发起请求（鉴权和路由框架层请求验证用户数据，或者当前页面的其他接口请求）响应401
5. 重写umi request拦截器（参照：[请求时token过期自动刷新token](https://segmentfault.com/a/1190000016946316)），拦截到401响应时获取当前页面地址是否存在cas ticket，若存在则暂停当前请求，发送/oauth/token登陆请求。其中token请求grant_type为cas_ticket，必要参数ticket为从前端query中获取的ticket
6. 成功获取到access_token即为登陆成功，在前端的登陆成功后续处理中重新获取并刷新用户数据、重新发送刚才401的请求（注意原地址和原参数）

此时原前后端完全分离的系统成功接入cas登陆

# 退出登录
退出登录由前端自主实现，后端无需参与改造
用户点击退出登录，前端调用delete token接口，并删除localStorage中的token缓存，然后跳转到cas登出地址：https://sso-server/cas/logout
此时系统内部和cas的登陆状态均失效，退出cas单点登录

# 时序图
登陆的主要时序图如下：
![前后端分离接入CAS](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/04/10/qian-hou-duan-fen-li-jie-rucas.jpg)

以上，基于此方式接入cas的原理可应用于大部分前后端分离接入其他认证方式，例如接入手机验证码等
