
---
title: C语言STRSTR实例
date: 2017-11-25 08:12:03
categories: C
tags:
- HEAD_FIRST_C
---
## 知识点：
`strstr()`函数会在第一个字符串中查找第二个字符串，如果找到，他会返回第二个字符串在存储器中的位置。

<!-- more -->

{% codeblock lang:c %}
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

//head frist c page 94

char tracks [][80] = {
		"给我一首歌的时间",
		"七里香",
		"威廉古堡",
		"简单爱",
		"东风破",
		"叶惠美",
};

void find_track(char search_for[]) {
	int i;
	for (i = 0; i < 6; i++) {
		if (strstr(tracks[i], search_for)) {
			printf("find track %i:%s\n", i,tracks[i]);
		}else{
			// printf("your search is %s,not in %s\n",search_for,tracks[i]);
		}	
	}
}

int main()
{
	char search_for[80];
	printf("search for :");
	fgets(search_for, 80, stdin);

	//由于fgets接收有换行符号+\0，所以把换行符换成结束符
	search_for[strlen(search_for)-1]='\0';
	find_track(search_for);
	return 0;	
}

{% endcodeblock %}

## 结果：

```
search for :爱
find track 3:简单爱
请按任意键继续. . .

```
