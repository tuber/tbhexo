---
title: 字符指针与字符数组
date: 2017-11-20 08:12:03
categories: C
tags:
- HEAD_FIRST_C
---

{% codeblock lang:c %}
int main()
{
	char masked_raider[] = "Alived";

	char *jimmy = masked_raider;

	printf("masked_raider is %s,jimmy is %s\n", masked_raider, jimmy);

	masked_raider[0] = 'd';
	masked_raider[1] = 'e';
	masked_raider[2] = 'a';
	masked_raider[3] = 'd';
	masked_raider[4] = 'e';
	masked_raider[5] = 'd';
	printf("masked_raider is %s,jimmy is %s\n", masked_raider, jimmy);
	
    return 0;
}
{% endcodeblock %}
<!-- more -->

## 知识点：

	jimmy和masked_raider 是一个存储器地址的两个别名
	字符串字面值保存在只读存储器中
	如果要修改字符串，需要在新的数组中创建副本

	char masked_raider[] = "Alived";

	程序会在栈上创建一个maskd_raider的数组，并把右值设置为 Alived

	
	char *jimmy ="Alived";

	程序会把常量值放在常量存储区，常量存储期是只读的。
	接着在栈上创建了jimmy变量，是局部变量
	然后把栈上的jimmy变量设置为alived的地址
	const char *jimmy = "Alived";
	jimmy[0] = 'd'
	因为常量存储区是只读的，所以用jimmy[0]='d'的时候，会报错，jimmy为不可修改的左值
