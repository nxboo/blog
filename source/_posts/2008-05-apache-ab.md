---
layout:     post
title:      "Apache ab 中文注释"
date:       2015-05-21
author:     "Bin"
tags:
    - apache
    - ab
---
This is ApacheBench, Version 2.0.40-dev <$Revision: 1.146 $> apache-2.0
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Copyright 1997-2005 The Apache Software Foundation, http://www.apache.org/

Benchmarking www.google.com (be patient).....done


Server Software:        GWS/2.1
Server Hostname:        www.google.com
Server Port:            80

Document Path:          /
Document Length:        230 bytes

Concurrency Level:      10
/*整个测试持续的时间*/
Time taken for tests:   3.234651 seconds
/*完成的请求数量*/
Complete requests:      10
/*失败的请求数量*/
Failed requests:        0
Write errors:           0
Non-2xx responses:      10
Keep-Alive requests:    10
/*整个场景中的网络传输量*/
Total transferred:      6020 bytes
/*整个场景中的HTML内容传输量*/
HTML transferred:       2300 bytes
/*大家最关心的指标之一，相当于 LR 中的 每秒事务数 ，后面括号中的 mean 表示这是一个平均值*/
Requests per second:    3.09 [#/sec] (mean)
/*大家最关心的指标之二，相当于 LR 中的 平均事务响应时间 ，后面括号中的 mean 表示这是一个平均值*/
Time per request:       3234.651 [ms] (mean)
/*这个还不知道是什么意思，有知道的朋友请留言，谢谢 ^_^ */
Time per request:       323.465 [ms] (mean, across all concurrent requests)
/*平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题*/
Transfer rate:          1.55 [Kbytes/sec] received
/*网络上消耗的时间的分解，各项数据的具体算法还不是很清楚*/
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       20  318 926.1     30    2954
Processing:    40 2160 1462.0   3034    3154
Waiting:       40 2160 1462.0   3034    3154
Total:         60 2479 1276.4   3064    3184

/*下面的内容为整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间，其中 50％ 的用户响应时间小于 3064 毫秒，60 ％ 的用户响应时间小于 3094 毫秒，最大的响应时间小于 3184 毫秒*/
Percentage of the requests served within a certain time (ms)
  50%   3064
  66%   3094
  75%   3124
  80%   3154
  90%   3184
  95%   3184
  98%   3184
  99%   3184
100%   3184 (longest request)

-n requests     Number of requests to perform
//在测试会话中所执行的请求个数。默认时，仅执行一个请求
-c concurrency Number of multiple requests to make
//一次产生的请求个数。默认是一次一个。
-t timelimit    Seconds to max. wait for responses
//测试所进行的最大秒数。其内部隐含值是-n 50000。它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。
-p postfile     File containing data to POST
//包含了需要POST的数据的文件.
-T content-type Content-type header for POSTing
//POST数据所使用的Content-type头信息。
-v verbosity    How much troubleshooting info to print
//设置显示信息的详细程度 - 4或更大值会显示头信息， 3或更大值可以显示响应代码(404, 200等), 2或更大值可以显示警告和其他信息。 -V 显示版本号并退出。
-w              Print out results in HTML tables
//以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。
-i              Use HEAD instead of GET
// 执行HEAD请求，而不是GET。
-C attribute    Add cookie, eg. 'Apache=1234. (repeatable)
//-C cookie-name=value 对请求附加一个Cookie:行。 其典型形式是name=value的一个参数对。此参数可以重复。
-P attribute    Add Basic Proxy Authentication, the attributes
                are a colon separated username and password.
//-P proxy-auth-username:password 对一个中转代理提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。

