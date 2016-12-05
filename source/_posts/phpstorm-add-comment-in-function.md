---
title: PHPSTORM中给一个方法动态添加注释
date: 2016-12-05 13:39:22
categories: PHP
tags:
 - PHPSTORM
 - PHP
 - 编辑器
---

## 目的：在phpstorm中，动态的给一个方法添加注释 ##


## 先添加一个动态模版【Live Templates】

*ct==Crate Time*

![添加动态模版][1]


[1]: /img/php/storm-1.jpg

## 编辑这个动态模版【Live Templates】，并应用（Application）到 PHP Comments，可见图一第五步
![编辑动态模版][2]

[2]: /img/php/storm-2.jpg

## 最后在注释生成的部分加载这个动态模版

![加载动态模版][3]

[3]: /img/php/storm-3.jpg

## EG

```
 /**
     * Desc:看房团热门路线推荐
     * v4.1.7新增接口
     * @Date&Time 2016-12-01 16:48
     * User: TongBo
     */
```


