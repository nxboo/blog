---
layout:     post
title:      "nginx fastcgi cache"
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

# http层

```conf
fastcgi_cache_path /tmp/nginx/fcgi/cache levels=1:2 keys_zone=fcgi:10m inactive=30m max_size=128m;  
fastcgi_cache_key  "$scheme$request_method$host$request_uri";  
fastcgi_temp_path  /tmp/nginx/fcgi/temp;  
```

fastcgi_temp_path: 生成fastcgi_cache临时文件目录  

fastcgi_cache_path: fastcgi_cache缓存目录，可以设置目录哈希层级，比如2:2会生成256*256个字目录，keys_zone是这个缓存空间的名字，cache是用多少内存（主要缓存key和文件元信息，不会缓存页面），inactive表示默认失效时间，max_size表示最多用多少硬盘空间，需要注意的是fastcgi_cache缓存是先写在fastcgi_temp_path再移到fastcgi_cache_path，所以这两个目录最好在同一个分区，从0.8.9之后可以在不同的分区，不过还是建议放同一分区。  

fastcgi_cache_key: 定义fastcgi_cache的key，示例中就以请求的(get\post)+host+URI作为缓存的key，Nginx会取这个key的md5作为缓存文件，如果设置了缓存哈希目录，Nginx会从后往前取相应的位数做为目录。



# server/location层

在server层中的location中做如下配置既可针对特定的路由进行设置

include backendServers/fcgi_cache_params;
fastcgi_cache_valid 200 302 1s;
fcgi-cache

```nginx
fastcgi_cache fcgi;
fastcgi_cache_valid 200 302 1s;
fastcgi_cache_valid 404 500 502 503 504 0s;
fastcgi_cache_valid any 1m;
fastcgi_cache_min_uses 1;
fastcgi_cache_use_stale error timeout invalid_header http_500 http_503 updating;
add_header X-Cache "$upstream_cache_status - $upstream_response_time";
fastcgi_cache_key "$scheme$request_method$host$request_uri";
```

* fastcgi_cache：用哪个缓存空间（在http层中我们定义的keys_zone）

* fastcgi_cache_valid：定义哪些http头要缓存 示例中200 302 都是1s 缓存 404 500等 0s缓存 其他的状态是1分钟缓存

* fastcgi_cache_use_stale：定义哪些情况下用过期缓存

* fastcgi_cache_min_uses：URL经过多少次请求将被缓存

配置这些参数时，注意每个参数的作用域，像fastcgi_cache_path参数，只能在http配置项里配置，而fastcgi_cache_min_uses这个参数，可以在http、server、location三个配置项里配置。这样更灵活的会每个域名、每个匹配的location进行选择性cache了。具体的参数作用域，参考FASTCGI模块的官方WIKI。我为了调试方便，添加了一个『X-Cache』的http响应头，$upstream_cache_status 变量表示此请求响应来自cache的状态，$upstream_response_time为过期时间 $upstream_cache_status


# 几种状态分别为：

|状态|说明|
|----|---|
| MISS | 未命中 |
| EXPIRED | expired, request was passed to backend Cache已过期 |
| UPDATING | expired, stale response was used due to proxy/fastcgi_cache_use_stale updating Cache已过期，(被其他nginx子进程)更新中 |
| STALE | expired, stale response was used due to proxy/fastcgi_cache_use_stale Cache已过期，响应数据不合法，被污染HIT 命中cache |

有一些情况会影响到cache的命中 这里需要特别注意

Nginx fastcgi_cache在缓存后端fastcgi响应时，当响应里包含“set-cookie”时，不缓存;
当响应头包含Expires时，如果过期时间大于当前服务器时间，则nginx_cache会缓存该响应，否则，则不缓存;
当响应头包含Cache-Control时，如果Cache-Control参数值为no-cache、no-store、private中任意一个时，则不缓存，如果Cache-Control参数值为max-age时，会被缓存，且nginx设置的cache的过期时间，就是系统当前时间 + mag-age的值。



//逐个测试，测试时，注释其他的
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

PHP
``` php
session_cache_limiter("none");
session_start();
echo date("Y-m-d H:i:s",time());
exit;
```

nginx中还有很多有趣且巧妙的用法 这里就不多讲了 最后贴一份自己的conf的location部分

```
location ~ \.php($|/) {
        fastcgi_pass unix:/tmp/php53-cgi.sock;
        include backendServers/fastcgi_params;
        fastcgi_split_path_info         ^(.+\.php)(/.+)$;
        fastcgi_param PATH_INFO         $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;

        set $xwsoul_nocache yes; #默认关闭cache
        set $cache_time 2;

        if ($request_uri ~* "^/gamespace/.*"){
                set $xwsoul_nocache ""; # 开启cache
                set $cache_time 10;
        }

        if ($request_uri ~* "^/comIdentify/show/site/\d+/subject/\d+"){
                set $xwsoul_nocache "";
        }

         if ($cookie_u != '') {
                set $xwsoul_nocache yes; #登陆用户关闭cache
        }

        ### fcgi-cache
        fastcgi_cache_bypass $xwsoul_nocache;
        fastcgi_no_cache $xwsoul_nocache;

        fastcgi_cache_valid  $cache_time;
        include backendServers/fcgi_cache_params;

}
```
