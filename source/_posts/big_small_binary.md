---
title: 大小端字节序
date: 2017-11-04 21:12:03
categories: C
tags:
- 大端字节序
---

大端字节序：数据的低位字节保存在内存的高地址端，等于内存的低位保存的是数据的高位地址
小端字节序：数据的高位字节保存在内存的高地址端，
网络字节序：tcpip是基于大端字节序
 <!-- more -->

缘由：内存每个地址单元对应一个字节，就是8bit。现在为32或者64位cpu，寄存器的宽度在大于了一个字节之后，字节排放就需要有个顺序，就有了大端和小端。所以大端和小端指的是寄存器的排列顺序。所以只是在跨平台或者网络编程程序中会经常用到，一般情况不会用到。

在c语言中，指针大小在32位机器上为4字节*8bit，64位机器上为8字节*8bit

----------------------- 最高内存地址 0xffffffff
栈底
栈
栈顶
-----------------------

NULL (空洞)
-----------------------
堆
-----------------------
未初始 化的数据
----------------------- 统称数据段
初始化的数据
-----------------------
正 文段(代码段)
----------------------- 最低内存地址 0x00000000


内存低地址存数据的高位，内存的高地址存数据的低位。
内存是由低到高增长，按照单个进程内的存储器地址由低到高分别为 代码段 数据段（全局变量+静态变量） 堆（动态分配内存） 栈（局部变量）

如何验证本机是低位还是高位？不同的处理器有不同的大小端模式


本地a字节序转成网络字节序-发送给b  b接收网络字节序 -b转成本地字节序 ，网络字节序为大端模式


比如内存地址由低到高：0x01 0x02
数据为12345678，数据的1234为高位，5678为低位。

0x01存的是1234，内存低地址存数据高位，为大端。

{% codeblock lang:c %}
#include <stdio.h>

int main(){


  int num=0x12345678;

  char *pnum = (char *)&num;

  printf(" sizeof pnum is %i\n",sizeof(pnum));

  printf("first %p,value is %x\n",pnum,pnum[0]);

  printf("second %p,value is %x\n",pnum+1,pnum[1]);
  printf("third %p,value is %x\n",pnum+2,pnum[2]);
  printf("fourth %p,value is %x\n",pnum+3,pnum[3]);


}
{% endcodeblock %}


执行结果，可以看出，78存在了低地址位，所以是小端序。

·
```
sizeof pnum is 4
first 1703740,value is 78
second 1703741,value is 56
third 1703742,value is 34
fourth 1703743,value is 12
Press any key to continue
```

结合php的PACK/unpack，可以得出一致结果:小端序


```
<?php


define('BIG_ENDIAN', pack('L', 1) === pack('N', 1));

if (BIG_ENDIAN)
{
  echo "大端序";
}
else
{
  echo "小端序";
}
```
