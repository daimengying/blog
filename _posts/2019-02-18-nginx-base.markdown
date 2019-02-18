---
layout: post
title:  "nginx动静分离"
categories: Nginx
tags: Nginx
author: mydai
description: nginx配置静态资源与动态访问分离
---
### location 语法规则

```
location [=|~|~*|^~] /uri/ {
        ····· 
}
```

```
=     开头表示精确匹配

^~    开头表示uri以某个常规字符串开头，理解为匹配url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）

~     开头表示区分大小写的正则匹配

~*    开头表示不区分大小写的正则匹配

!~    区分大小写不匹配的正则

!~*   不区分大小写不匹配的正则

/     通用匹配，任何请求都会匹配到
```
当我们有多个 location 配置的情况下，其匹配顺序为：

首先匹配 "="，其次匹配 "^~", 其次是按文件中顺序的正则匹配，最后是交给 "/" 通用匹配。


```
当有匹配成功时候，停止匹配，按当前匹配规则处理请求。
location = / {
   #规则A
}
location = /login {
   #规则B
}
location ^~ /static/ {
   #规则C
}
location ~ \.(gif|jpg|png|js|css)$ {
   #规则D
}
location ~* \.png$ {
   #规则E
}
location !~ \.xhtml$ {
   #规则F
}
location !~* \.xhtml$ {
   #规则G
}
location / {
   #规则H
}
```
在实际应用中，至少需要有三个匹配规则定义，如下：

```
# 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
# 这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://localhost:8080/index
}

# 第二个必选规则是处理静态文件请求，这是 nginx 作为 http 服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}

# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器
# 非静态文件请求就默认是动态请求，自己根据实际把握
# 毕竟目前的一些框架的流行，带 .php, .jsp 后缀的情况很少了
location / {
    proxy_pass http://localhost:8080/
}
```
### rewrite 语法
last       – 基本上都用这个 Flag

break      – 中止 Rewirte，不在继续匹配

redirect   – 返回临时重定向的HTTP状态302

permanent  – 返回永久重定向的HTTP状态301

##### 可以用来判断的表达式
-f 和 !-f    用来判断是否存在文件

-d 和 !-d    用来判断是否存在目录

-e 和 !-e    用来判断是否存在文件或目录

-x 和 !-x    用来判断文件是否可执行

##### 可以用作判断的全局变量
例：http://localhost:88/test1/test2/test.php

   $host：localhost

   $server_port：88

   $request_uri：http://localhost:88/test1/test2/test.php

   $document_uri：/test1/test2/test.php

   $document_root：D:\nginx/html

   $request_filename：D:\nginx/html/test1/test2/test.php
   
