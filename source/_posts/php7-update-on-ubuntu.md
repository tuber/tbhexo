---
title: 在ubuntu上升级到PHP7
date: 2016-11-21 13:41:11
categories: PHP
tags:
 - PHP7
 - FPM
 - Ubuntu
---


## 卸载旧版本php ##

`apt-get autoremove php*`

## 安装新源 ##

```
apt-get install software-properties-common、
add-apt-repository ppa:ondrej/php
apt-get update
```

## 安装新版php ##

 <!-- more -->

```
apt-get install php-common php-cli php-fpm php-mysql php-gd php-dev php-zip php-pear php-curl php-mbstring

```

## 一些命令及路径 ##


`/etc/init.d/php7.0-fpm` 中的重启php7.0fpm命令

```
root@lyh:/etc/init.d# /etc/init.d/php7.0-fpm status[start stop]
php7.0-fpm start/running, process 1704

```


- `/etc/php/7.0/fpm/php-fpm.conf`中的配置

  `pid = /run/php/php7.0-fpm.pid`

- `/etc/php/7.0/fpm/pool.d/www.conf`中的

  `listen = /run/php/php7.0-fpm.sock`

- `/etc/nginx/sites-enabled`中的yoursite.conf，和上面路径一致

  `fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;`

## PHP-v ##

```
root@lyh:/etc/php/7.0/fpm# php -v
PHP 7.0.13-1+deb.sury.org~trusty+1 (cli) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.13-1+deb.sury.org~trusty+1, Copyright (c) 1999-2016, by Zend Technologies

```
