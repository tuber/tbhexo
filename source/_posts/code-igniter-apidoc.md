---
title: 在CI框架中加入生成api文档
date: 2017-01-20 14:18:59
categories: PHP
tags:
 - PHP框架
 - CI框架
 - 接口文档
---

先上图
---
**文档列表页面-1**

![图片描述][1]

**文档列表页面-2**

<!-- more -->

![图片描述][2]

**文档详情页面**

![图片描述][3]


主要是抽取自[phalapi][4]
------------------

使用方法
----
1. 如果使用默认ci框架及结构目录，只需将`Controller/doc.php`,`views/doc/*`的两个模版文件放入项目即可。
2. 如果其他项目引入，只需在`Controller/doc.php`中指定项目Controller目录，以及对应的文件夹名对应的类名方法即可。
3. 文档注释方法可以参考代码中`doc.php`中的注释


[GITHUB下载地址][5]


  [1]: /img/php/api-doc-1.jpg
  [2]: /img/php/api-doc-2.jpg
  [3]: /img/php/api-doc-3.jpg
  [4]: http://www.phalapi.net/
  [5]: https://github.com/tuber/apidoc
