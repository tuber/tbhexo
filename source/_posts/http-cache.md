---
title: HTTP-CAHCE中的CACHE-CONTROL、Etag、Modify
date: 2017-03-07 13:41:11
categories: PHP
tags:
 - HTTP
 - CACHE
 - PHP
 - NGINX
---

用世界上最好的语言演示一下etag
-------------

```
<?php

// apache 服务器，如果您是nginx请自行配置读取header等信息，同时下面会有nginx测试

$file = 'etag.txt';
$etag = md5_file($file);
$headers = apache_request_headers();

if (isset($headers['If-None-Match']) && trim($headers["If-None-Match"]) == $etag) {
    header("HTTP/1.1 304 Not Modified");
} else {
    $content = file_get_contents($file);
    header("Etag: $etag");
    echo $content;
}

```
第一次请求，服务器返回200.我分别列下请求头【RequsetHeaders】和响应头【ResponseHeaders】

请求头
```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Cache-Control:no-cache
Connection:keep-alive
Host:kache.com
Pragma:no-cache
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.
```
响应头

```
Request URL:http://kache.com/etag.php
Request Method:GET
Status Code:200 OK
Remote Address:127.0.0.1:80

Connection:Keep-Alive
Content-Type:text/html
Date:Tue, 07 Mar 2017 13:02:13 GMT
Etag:966aa4bd5183fd9358fd222647c5c6a3
Keep-Alive:timeout=5, max=99
Server:Apache/2.4.10 (Win32) OpenSSL/0.9.8zb mod_fcgid/2.3.9
Transfer-Encoding:chunked
X-Powered-By:PHP/5.4.33
```
需要注意第一次请求头没有If-None-Match:，注意第一次响应头有Etag:这个标签,注意第一次是200


----------


第二次请求

请求头：

```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch
Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
Cache-Control:max-age=0
Connection:keep-alive
Host:kache.com
If-None-Match:966aa4bd5183fd9358fd222647c5c6a3
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36
```

响应头:

```
Request URL:http://kache.com/etag.php
Request Method:GET
Status Code:304 Not Modified
Remote Address:127.0.0.1:80

Connection:Keep-Alive
Date:Tue, 07 Mar 2017 13:02:16 GMT
Keep-Alive:timeout=5, max=98
Server:Apache/2.4.10 (Win32) OpenSSL/0.9.8zb mod_fcgid/2.3.9
```
需要注意第二次请求头有If-None-Match:，注意第二次响应为304

----------

再用php语言演示一下Last-Modified
------------------------

```
<?php
/**
 *If-Modified-Since[request] & Last-Modified [response]
 *减少网络字节传输
 */
$headers = apache_request_headers();
$file = 'modified.txt';

if (isset($headers['If-Modified-Since']) && (strtotime($headers['If-Modified-Since']) == filemtime($file))) {

    header('Last-Modified: ' . gmdate('D, d M Y H:i:s', filemtime($file)) . ' GMT', true, 304);

} else {
    header('Last-Modified: ' . gmdate('D, d M Y H:i:s', filemtime($file)) . ' GMT', true, 200);

    $content = file_get_contents($file);
    echo $content;
}

```
If-Modified-Since[请求头的] 和 Last-Modified [响应头的]这一对关系的HADER头我这里就不贴了。稍后的ppt里有比较详细的说明

----------

我们常用的就是Last-Modified和Etag一起来用，上php代码
------------------------------------

```
<?php

$file='etag_modify.txt';

$last_modified_time=filemtime($file);

$etag = md5_file($file);

$headers = apache_request_headers();


if (@strtotime($headers['If-Modified-Since']) == $last_modified_time ||
    @trim($headers['If-None-Match']) == $etag) {
  header('Content-Length: '.filesize($file));
    header("HTTP/1.1 304 Not Modified");

}else{

  header("Etag: $etag");
  header('Last-Modified: '.gmdate('D, d M Y H:i:s', $last_modified_time).' GMT', true, 200);
  header('Content-Length: '.filesize($file));
  $content=file_get_contents($file);
  echo $content;

}

```
这样做的好处是双重验证，同时满足两者条件才会缓存失效，弥补了modify的粒度最多为秒的问题以及modify的打开关闭即更改时间的问题。当然etag也会有坑，不同物理机可能会导致相同文件不同结果（没实验过）


----------

直接上NGINX配置示例
------------
毕竟php做服务端水平有限，大家可以参考  swoole framework OR workerman中对etag和modify的处理。

不多说，上NGINX配置段，为了演示modify ，可以在/etc/nginx/nginx.conf中把etag关闭

```
http {

        ##
        # Basic Settings
        ##
        etag  off;
        ...
```

关于其他静态文件缓存的设置

```
location ~* \.(?:css)$ {
  #expires 1y;
  add_header  Cache-Control max-age=5;
  add_header Cache-Control "public";
  add_header  Last-Modified "";
}

```
简单对以上括号内代码说明：
1. expires 1y; 是http协议1.0写法，1.1对应的是cache-contorl:max-age='';前者为GMT绝对时间，后者为相对时间。
2. add_header  Cache-Control max-age=5; 缓存5秒，如果没有Last-Modified（即设置了 add_header  Last-Modified "";） ，期间会一直直接请求服务器，服务器一直返回200，如果有设置Last-Modified，5秒后会请求一次服务器，5秒前会返回304.

简单归纳Last-Modified和max-age（expires）关系

1. 如果设置了max-age=0，而没有启用modify，那么不会缓存

2. 如果单单启用modify，而没有max-age==0.也会缓存

3. 如果启用了modify，并且 max-age=0，那么不会缓存

4. 如果设置了max-age=1000，但没有启用modify ，不会缓存

再此说明上面配置导致的结果：5秒内如果文件有变化，那么客户端不会有任何感知。5秒后将会重新发起请求，得到200响应。然后再缓存5秒【注意没有开启etag】

----------
下面这个例子和上面一样，是针对图片等，缓存1一个月，即使服务端删除了，1个月内也会正常显示（除非ctrl+f5，或者服务端重启了）public代表任何代理服务器都可以缓存，对应的为private，只允许客户端浏览器缓存。

```
location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc)$ {
  expires 1M;
  add_header Cache-Control "public";
  add_header  Last-Modified "";
}

```
----------

那需要有更改就更新，怎么办？
--------------

```
location ~* \.(?:css)$ {
  expires 1y;
 # add_header  Cache-Control max-age=5;
 # add_header Cache-Control "public";
 # add_header  Last-Modified "";
}
```
以上配置虽然过期时间是一年，但是服务端会返回Last-Modified，来确认，意思为你就vim了一下xx.css,即使没有做任何更改，浏览器也会重新发起请求。你要是没改，那八成就一年后见了。

那我就不想缓存怎么办？
-----------
用cache-control：控制
用no-cahce【浏览器端等可以缓存，但是没有什么卵用】
用no-store【浏览器端等不用缓存，不用费劲。每次都跟我服务端请求】
用must-revalidate，浏览器端别整没用的，到期了就马上跟我请求。麻溜的必须。防止的就是代理服务器等自作聪明，认为没有过期。

```
location ~* \.(?:js)$ {
  add_header  Cache-Control no-cache;
  add_header  Cache-Control must-revalidate;
}
```

[测试代码和PPT在这里][1]
-----------


  [1]: https://github.com/tuber/HTTP_CACHE
