
---
title: C语言str_reverse实例
date: 2017-11-25 08:25:03
categories: C
tags:
- HEAD_FIRST_C
---
## 知识点：
字符数组的字符串指针反转
<!-- more -->

{% codeblock lang:c %}
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

//head frist c page 98

void print_reverse(char *s) {

	size_t len = strlen(s);
	//指针相加，t为s对应指针的最后一个
	char *t = s + len - 1;

	//最后一个肯定大于第一个s指针指向的位置，
	while (t >= s) {
		printf("%c",*t);
		t = t - 1;
	}
	//puts("") 类似于printf("%s\n",s);pust在字符末尾会自动输出一个回车符
	puts("我隔开了");
	puts ("");
}
int main()
{
	char letter[] = "abcd";
	print_reverse(letter);
	return 0;
}

{% endcodeblock %}

## 结果：

```
dcba我隔开了

请按任意键继续. . .

```
