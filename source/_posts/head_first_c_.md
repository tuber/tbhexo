---
title: 字符数组+1与指针+1
date: 2017-11-04 21:12:03
categories: C
tags:
- 数组与指针
---



## 字符数组+1代码

{% codeblock lang:c %}
#include <stdio.h>

int main(){

  char mystr[]="Abcef";
  printf("size of mystr=%i\n",sizeof(mystr));

  printf("mystr[0]，他的值为%c:\n",mystr[0]);
  printf("&mystr[0]，他的内存地址值为%i:\n",&mystr);

  printf("mystr[1]，他的值为%c:\n",mystr[1]);
  printf("&mystr[1]，他的内存地址值为%i:\n",&mystr+1);

  printf("mystr[2]，他的值为%c:\n",mystr[2]);
  printf("&mystr[2]，他的内存地址值为%i:\n",&mystr+2);

  printf("mystr[3]，他的值为%c:\n",mystr[3]);
  printf("&mystr[3]，他的内存地址值为%i:\n",&mystr+3);

  return 0;
}
{% endcodeblock %}
 <!-- more -->

## 字符数组+1代码 结果

```
size of mystr=6
mystr[0]，他的值为A:
&mystr[0]，他的内存地址值为1703736:
mystr[1]，他的值为b:
&mystr[1]，他的内存地址值为1703742:
mystr[2]，他的值为c:
&mystr[2]，他的内存地址值为1703748:
mystr[3]，他的值为e:
&mystr[3]，他的内存地址值为1703754:
Press any key to continue
```

## 指针+1代码

{% codeblock lang:c %}

#include <stdio.h>

int main(){
  char *mystr="ABC";
  printf("size of mystr=%i\n",sizeof(mystr));
  printf("mystr[0]，他的值为%c:\n",mystr[0]);
  printf("&mystr[0]，他的内存地址值为%i:\n",&mystr);
  printf("mystr[1]，他的值为%c:\n",mystr[1]);
  printf("&mystr[1]，他的内存地址值为%i:\n",&mystr+1);
  printf("mystr[2]，他的值为%c:\n",mystr[2]);
  printf("&mystr[2]，他的内存地址值为%i:\n",&mystr+2);
  printf("mystr[3]，他的值为%c:\n",mystr[3]);
  printf("&mystr[3]，他的内存地址值为%i:\n",&mystr+3);
  return 0;
}
{% endcodeblock %}
## 指针+1代码 结果
```
size of mystr=4
mystr[0]，他的值为A:
&mystr[0]，他的内存地址值为1703740:
mystr[1]，他的值为B:
&mystr[1]，他的内存地址值为1703744:
mystr[2]，他的值为C:
&mystr[2]，他的内存地址值为1703748:
mystr[3]，他的值为
```
## 结论：
指针的+1，只对当前指针所占字节数的+1
字符数组的+1，是对当前数组指针的sizeof+1
