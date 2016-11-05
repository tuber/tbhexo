---
title: 硬盘中磁道、扇区基本概念
date: 2016-11-05 18:02:33
categories: computer
tags:
- 硬盘
- 扇区
- 磁道

---





用`AIDA64 Extreme`工具看下我的low硬盘
--------------------------

![图片描述][1]

柱面磁头扇区磁道？
---------

 *WTF*?

 <!-- more -->


一图胜千言
-----
![磁盘2][2]


![磁盘3][3]

 在看个二合一版的图

![磁盘4][4]

温习下英语
-----

  -  磁头（head）
  -  磁道就是一个圈（track）
  -  柱面就是多个磁道号相同的圈组成的（cylinder）
  -  扇区（sector）
  -  圆盘（platter）


再回到我的low硬盘
----------

 存储容量 ＝ 磁头数 × 磁道(柱面)数 × 每道扇区数 × 每扇区字节数

 `248085*63*16*512/1000/1000 = 128 035.67616M`

 注意512是Byte

简单说一下
-----

 磁盘由N个盘片构成，每个盘片一般有两面，一面一个磁头，两面都可以存储数据

 磁道号相同的组成一个柱面，柱面是我们硬盘分区时候最小单位。

 sector 扇区 磁道按512Byte分成若干扇区，计算机对硬盘读写，是按扇区为最小单位。
 而一般文件系统中的BLOCK为KB，通常为4KB.（现在有的硬盘每个扇区有4K了）

 可以这么说：即使读一个字节，也必须把这512字节全部读入内存

在linux上看一把硬盘
------------

     root@lyh:~# fdisk -l
     Disk /dev/xvda: 42.9 GB, 42949672960 bytes
     255 heads, 63 sectors/track, 5221 cylinders, total 83886080 sectors

  硬盘容量就是

`heads*sectors*cylinders*512=255*63*5221*512/1000/1000/1000 = 42.94418688G`

  [1]: /img/computer/cipan1.png
  [2]: /img/computer/cipan2.gif
  [3]: /img/computer/cipan3.gif
  [4]: /img/computer/cipan4.png
