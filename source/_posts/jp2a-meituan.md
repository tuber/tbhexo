---
title: jp2a把图片转为ascii
date: 2016-11-21 10:05:49
categories: LINUX
tags:
 - jp2a
 - ascii
 - meituan
---

## 先看效果 ##
<iframe height=498 width=510 src='http://player.youku.com/embed/XMTgyODIyMjA3Mg==' frameborder=0 'allowfullscreen'></iframe>

<!-- more -->

## 再贴代码 ##

```
#!/bin/bash
make
jp2a --color --background=light -b -f --term-fit  waimai.jpg >waimai.txt
cat waimai.txt
sleep 2
make clean >/dev/null 2>&1
jp2a --color --background=light -b -f --term-fit zaiyiqilvse.jpg >zaiyiqi.txt
cat zaiyiqi.txt

```

## 注意 ##

需要先安装jp2a，开头那些编译的效果是在编译一个php的扩展，和本身效果呈现没有关系。就是为了小装b用的
上面的sh脚本 和你需要编译的扩展在一起就行。



