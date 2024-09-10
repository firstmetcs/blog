---
title: Nginx去掉前端配置的路径前缀
excerpt: 在前端项目部署到Nginx的时候，前端访问后端的路径加了一个全局前缀，一般来说这是前后端商量好的，但是也会有不一般的情况，后端在遇到这种情况的时候有两种选择
date: 2024-09-10 11:00:00
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/redirect-url-in-nginx.png
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/nginx-rewrite-lg.jpeg
tags:
- Nginx
categories:
- Nginx
---

在前端项目部署到Nginx的时候，前端访问后端的路径加了一个全局前缀，一般来说这是前后端商量好的，但是呢也会有不一般的情况，后端在遇到这种情况的时候有两种选择
* 第一种是给项目里也加上同样的前缀
* 第二种就是Nginx配置文件做出修改

这里我选择了后者

1. 第一种方式，这样的结果是你后端项目同样需要加上`prod-api`
```conf
location /prod-api{
    proxy_pass http://193.1.0.6:10004;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
}
```

1. 第二种方式，直接prod-api 后加个斜杠，同时在端口后面也加个斜杠，这样实际请求的路径就去掉了`/prod-api`
这样访问的`/prod-api/test/1` 实际就会变成 `http://193.1.0.6:10004/test/1`，就像是springmvc的静态资源映射一样的
```conf
location /prod-api/{
    proxy_pass http://193.1.0.6:10004/;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
}
```
proxy_pass的结尾有/， 则会把`/prod-api/*`后面的路径直接拼接到后面，即移除`prod-api`.


3. 另一种方案是使用rewrite
```conf
location ^~/prod-api/ {
    proxy_set_header Host $host;
    proxy_set_header  X-Real-IP        $remote_addr;
    proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;

    rewrite ^/prod-api/(.*)$ /$1 break;
    proxy_pass http://service;
}
```
注意到proxy_pass结尾没有/， rewrite重写了url。
关于rewrite
```
syntax: rewrite regex replacement [flag]
Default: —
Context: server, location, if
```


注意点：

* 在proxy_pass 反向代理地址最后加一个/
* 在location匹配的url路径前添加^~/

location ^~/prod-api/：匹配任何以 /prod-api/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条


{% note info %}
本文参考自以下博文，版权归原作者所有。

原文作者：你是理想
原文地址：[nginx去掉前端配置的路径前缀](https://sspai.com/post/73588)

原文作者：Ryan Miao
原文地址：[Nginx代理proxy pass配置去除前缀](https://www.cnblogs.com/woshimrf/p/nginx-proxy-rewrite-url.html)
{% endnote %}