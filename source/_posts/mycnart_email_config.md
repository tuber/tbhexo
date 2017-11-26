
---
title: mycncart email设置
date: 2017-11-25 20:25:03
categories: PHP
tags:
- mycnart
---
## 知识点：查看log

提交订单时，出现服务器500错误，但是php的相关报错都已经打开。
想了一会，还是看`fpm_error.log`和`nginx errorlog`
<!-- more -->

果真，
`root@tuan:/var/log/nginx# tail -f error.log`

nginx log 如下

```
...
/php7.0-fpm.sock:", host: "www.maipingzheng.com", referrer: "http://www.maipingzheng.com/index.php?route=checkout/checkout"
2017/11/25 19:50:49 [error] 6104#6104: *504 FastCGI sent in stderr: "PHP message: PHP Fatal error:  Uncaught Exception: Error: EHLO not accepted from server! in /usr/share/nginx/cart_2.0.0.3/system/library/mail/smtp.php:120
Stack trace:
...

```

重点是 
`EHLO not acceptedfromserver!`，追了下代码发现原来是源自于smtp的错误，由于发送邮箱用的是腾讯的企业邮箱，而smtp.php没有涉及到ssl相关。


## 解决办法
smtp服务器由
`smtp.exmail.qq.com`
改为

`ssl://smtp.exmail.qq.com`


## 备忘~
opencart nginx url seo 配置
```
	server {

         location / {

                 try_files $uri @opencart;

         }

         location @opencart {

                 rewrite ^/(.+)$ /index.php?_route_=$1 last;

         }


         location ~* (\.(tpl|ini))$ {

                 deny all;

 		}
```

