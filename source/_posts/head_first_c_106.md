---
title: C根据标准输入解析为json
date: 2017-11-26 15:25:03
categories: C
tags:
- HEAD_FIRST_C
---


## 知识点：
标准输入读入以及`scanf`返回值及其参数的使用
<!-- more -->


## 原书版本
{% codeblock lang:c %}
// volvo1.cpp: 定义控制台应用程序的入口点。
#include "stdafx.h"
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
//head frist c page 106
int main() {
	float latitude;
	float longitude;
	char info[80];
	int started = 0;

	puts("data=[");
	while (scanf("%f,%f,%79[^\n]", &latitude, &longitude, info) == 3) {
		if (started) {
			printf(",\n");
		}
			
		else {
			started = 1;
		}
			
		printf("{latitude: %f, longitude: %f, info: '%s'}", latitude, longitude, info);
	}
	puts("\n]");
	return 0;
}
{% endcodeblock %}

## 结果：

```
data=[
12321.123,21321.123,asdffd
{latitude: 12321.123047, longitude: 21321.123047, info: 'asdffd'}12312.12,213123.213,sdafsdf
,
{latitude: 12312.120117, longitude: 213123.218750, info: 'sdafsdf'}^D

]
请按任意键继续. . .

```

## 我的版本：
{% codeblock lang:c %}
// volvo1.cpp: 定义控制台应用程序的入口点。


#include "stdafx.h"
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
//head frist c page 106
int main() {

	char name[40];
	char album[40];
	int year;
	char comments[90];
	int startd = 0;
	puts("data=[");
	while (scanf("%[^,], %[^,], %i, %89[^\n]", name, album, &year, comments) == 4) {
		if (startd) {
			printf("，\n");
		}
		else {
			startd = 1;
		}
	   printf("{歌曲名称:'%s', 所属专辑：'%s',出品日期:%i，歌词: '%s'}", name, album, year, comments);
	}
	puts("\n]");
	return 0;
}

{% endcodeblock %}

## 结果：

```
data=[
简单爱,范特西,2012,我想就这样...
{歌曲名称:'简单爱', 所属专辑：'范特西',出品日期:2012，歌词: '我想就这样...'}简单爱,范特西,2012,我想就这样...
，
{歌曲名称:'
简单爱', 所属专辑：'范特西',出品日期:2012，歌词: '我想就这样...'}简单爱,范特西,2012,我想就这样...
，
{歌曲名称:'

简单爱', 所属专辑：'范特西',出品日期:2012，歌词: '我想就这样...'},,

]
请按任意键继续. . .

```

## 结论：

scanf参数真多。。先体会体会~


{% codeblock lang:c %}
#include <stdio.h>  
bool skip(){  
    scanf("%*[^0-9]");  
    return true;  
}  
  
int main()  
{  
    int n;  
    while(skip() && scanf("%d", &n)!=EOF)  
        printf("%d\n", n);  
    return 0;  
}  
{% endcodeblock %}






