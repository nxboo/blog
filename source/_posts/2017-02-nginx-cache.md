---
layout:     post
title:      "Nginx fastcgi cache"
subtitle:   "The Next Generation Application Model For The Web - Progressive Web App"
date:       2017-02-28
author:     "Hux"
header-img: "img/post-bg-nextgen-web-pwa.jpg"
header-mask: 1
catalog:    true
tags:
    - nginx
    - cache
---
# 相关参数

## fastcgi_cache_path
> syntax: fastcgi_cache_path path [ levels = levels ] keys_zone = name : size [ inactive = time ] [ max_size = size ] [ loader_files = number ] [ > loader_sleep = time ] [ loader_threshold = time ]
> default: none
> context: http

```nginx
fastcgi_cache_path /dev/shm/nginx_cache levels=1:2 keys_zone=fcgicache:10m inactive=50m max_size=128m;
```
* /tmp/nginx/fcgi/cache 缓存目录
* keys_zone=fcgicache:10m keys_zone是这个缓存空间的名字，10m是用多少内存（主要缓存key和文件元信息，不会缓存页面）,这个会在fastcgi_cache中引用
* levels=1:2 设置目录哈希层级，比如2:2会生成256*256个字目录
* inactive 失效时间
* max_size 最多用多少空间


## fastcgi_temp_path
> Syntax:	fastcgi_temp_path path [ level1 [ level2 [ level3 ]]]
> Default:	fastcgi_temp
> Context:	http server location

生成fastcgi_cache临时文件目录，fastcgi_cache缓存是先写在fastcgi_temp_path再移到fastcgi_cache_path，所以这两个目录最好在同一个分区，从0.8.9之后可以在不同的分区，不过还是建议放同一分区


## fastcgi_cache
> Syntax:	fastcgi_cache zone | off;
> Default: off;
> Context:	http, server, location

用哪个缓存空间，fastcgi_cache_path定义

``` nginx
fastcgi_cache fcgicache;
```

> Syntax:	fastcgi_cache_key string;
> Default:	—
> Context:	http, server, location

设置缓存的key
```nginx
fastcgi_cache_key "$scheme$request_method$host$request_uri";
```
建议：在http层定义一个，如果在server或location要修改的必要，再进行修改


## fastcgi_cache_methods
> Syntax:	  fastcgi_cache_methods GET | HEAD | POST ...;
> Default:  GET HEAD;
> Context:  http, server, location

允许缓存的请求类型

## fastcgi_cache_min_uses

> Syntax:	fastcgi_cache_min_uses number;
> Default:  1;
> Context:	http, server, location

最少要被请求多少次才会缓存

## fastcgi_cache_use_stale
> syntax: fastcgi_cache_use_stale [error | timeout | invalid_header | updating | http_500 | http_503 | http_403 | http_404 | off]
> default: fastcgi_cache_use_stale off;
> context: http, server, location

定义哪些情况下用过期缓存
```nginx
fastcgi_cache_use_stale error timeout invalid_header http_500 http_503 updating;
```

## fastcgi_cache_valid
> Syntax:	fastcgi_cache_valid [code ...] time;
> Default:	—
> Context:	http, server, location

针对不同的状态，设置不同的缓存时间，可多行
```nginx
fastcgi_cache_valid 200 302  10m;
fastcgi_cache_valid 301      1h;
fastcgi_cache_valid any      1m;
fastcgi_cache_valid 1m;  #等同于 fastcgi_cache_valid 200 301 302 1m;
```


# 在响应头中设置缓存时间

缓存的参数也可以直接在响应头中设置。这比使用指令设置缓存时间具有更高的优先级。

* “X-Accel-Expires”标头字段设置响应的缓存时间（以秒为单位）。零禁用缓存。如果值以@前缀开始，则它设置以当前时间开始的以秒为单位的绝对时间，直到响应可以被缓存。
* 如果报头不包括“X-Accel-Expires”字段，则可以在报头字段“Expires”或“Cache-Control”中设置高速缓存的参数。
* 如果头包括“Set-Cookie”字段，则这样的响应将不被缓存。
* 如果头中Vary *，则这样的响应将不被缓存（1.7.7）。如果报头包括具有另一个值的“Vary”字段，则这样的响应将被缓存，考虑相应的请求报头字段（1.7.7）。

# 为什么没有被缓存？


* Nginx fastcgi_cache在缓存后端fastcgi响应时，当响应里包含“set-cookie”时，不缓存;
* 当响应头包含Expires时，如果过期时间大于当前服务器时间，则nginx_cache会缓存该响应，否则，则不缓存;
* 当响应头包含Cache-Control时，如果Cache-Control参数值为no-cache、no-store、private中任意一个时，则不缓存
* 头中Vary * (1.7.7)

可以使用fastcgi_ignore_headers指令禁用对这些响应头字段中的一个或多个的处理。

```nginx
fastcgi_ignore_headers Expires Cache-Control;
fastcgi_ignore_headers Set-Cookie; #如果加入这条，则后端无法设置cookie,session
```

# 配置实例

``` nginx
http{
  #...
  fastcgi_cache_path /dev/shm/nginx_cache levels=1:2 keys_zone=fcgicache:10m inactive=30m max_size=128m;
  fastcgi_cache_key  "$scheme$request_method$host$request_uri";
  fastcgi_temp_path  /dev/shm/nginx_tmp;
  #...

  server{
    #...
    location ~ \.php($|/) {
        #...
        fastcgi_cache fcgicache; #用哪个缓存空间
        fastcgi_cache_valid 1m; #200 301 302缓存1分钟
        fastcgi_cache_valid 404 500 502 503 504 0s; #404 500 502 503 504不缓存
        fastcgi_cache_valid any 3h; #其它缓存3小时
        fastcgi_cache_min_uses 1; #经过一次请求后缓存
        fastcgi_cache_use_stale error timeout invalid_header http_500 http_503 updating; #哪些情况下用过期缓存
        fastcgi_ignore_headers Expires Cache-Control; #忽略Expires Cache-Control头
        #...
    }
    #...
  }
}
```

# 调试

* $upstream_response_time为过期时间
* $upstream_cache_status 表示此请求响应来自cache的状态

## 缓存状态：
|状态|说明|
|----|---|
| MISS | 未命中 |
| EXPIRED | expired, request was passed to backend Cache已过期 |
| UPDATING | expired, stale response was used due to proxy/fastcgi_cache_use_stale updating Cache已过期，(被其他nginx子进程)更新中 |
| STALE | expired, stale response was used due to proxy/fastcgi_cache_use_stale Cache已过期，响应数据不合法，被污染HIT 命中cache |


# PHP

逐个测试，测试时，注释其他的
``` php
header("Expires: ".gmdate("D, d M Y H:i:s", time()+10000).' GMT');
header("Expires: ".gmdate("D, d M Y H:i:s", time()-99999).' GMT');
header("X-Accel-Expires:5"); // 5s
header("Cache-Control: no-cache"); //no cache
header("Cache-Control: no-store"); //no cache
header("Cache-Control: private"); //no cache
header("Cache-Control: max-age=10"); //cache 10s
setcookie('hello',"testaaaa"); //no cache
echo date("Y-m-d H:i:s",time());
exit;
```

程序调用session_start时，php的session拓展自己输出的。session.cache_limit参数来决定输出包含哪种Expires的header，默认是nocache，修改php.ini的session.cache_limit参数为“none”即可让session模块不再输出这些http 响应头。或在调用session_start之前，使用session_cache_limiter函数来指定下该参数值。那为什么要在使用session时，发Expires、Cache-Control的http response header呢？我猜测了下，需要session时，基本上是用户跟服务器有交互，那么，既然有交互，就意味着用户的每次交互结果也可能不一样，就不能cache这个请求的结果，给返回给这个用户。同时，每个用户的交互结果都是不一样的，nginx也就不能把包含特殊Cache-Control的个人响应cache给其他人提供了。  


非要用session_start 可以如下方式

``` php
session_cache_limiter("none");
session_start();
echo date("Y-m-d H:i:s",time());
exit;
```
