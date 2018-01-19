---
title: C语言基本函数练习
date: 2018-01-16 19:41:30
tags: C
categories: 
	- C语言练习
---


学习了C语言基本内容，根据课后习题做一些记录

<!-- more -->
### 基本函数练习

本题要求实现一个计算m到n(m<n),保证返回的是，m到n之间所有整数之和

函数接口定义：
> int sum(int m, int n)

其中m和n是用户传入的参数，保证有m<n,函数返回的是m到n之间所有整数的和。

```C
#include <stdio.h>
int sum(int, int);
int main(void) {
	int m, n;
	scanf("%d %d", &m, &n); // 获取用户输入
	printf("sum = %d \n", sum(m, n)); // 打印并调用sum函数获得返回值输出
	return 0;
}
int sum(int m, int n) { // 简单的循环相加
	int sum = 0;
	for (int i = m; i <= n; i++) {
		sum += i;
	}
	return sum; // 返回循环之后的总和
}

```

找两个数中最大者

本题要求对两个整数a和b，输出其中较大的数。

函数接口定义
> int max(int a,int b);
其中a和b是用户传入的参数，函数返回的是两者中较大的数。

```C
#include <stdio.h>

int max(int, int);
int main(void) {
	int a, b;
	scanf("%d %d", &a, &b);
	printf("max = %d \n", max(a, b));
	return 0;

}
int max(int a, int b) {
	if (a > b) { // if判断 如果a大返回a否则返回b
		return a;
	}
	else {
		return b;
	}
}
```

数字金字塔

本题要求实现函数输出n行数字金字塔。

函数定义
> void pyramid( int n )

其中n是用户传入的参数，为[1, 9]的正整数。要求函数按照如样例所示的格式打印出n行数字金字塔。注
意每个数字后面跟一个空格

```C
#include <stdio.h>
void pyramin(int n);
int main(void) {
	int n;
	scanf("%d", &n);
	pyramin(n);
	return 0;
}

void pyramin(int n) {
	int i, j; 
	for (i = 1; i <= n; i++) { // 外层循环，控制层级
		for (j = 0; j < n - i; j++) { // 内层循环首先输入几个空格
			printf(" ");
		}
		for (j = 0; j <= i; j++) { // 然后在空格后面输入值
			printf("%d ", i);
		}
		printf(" \n"); // 循环完毕输出换行符
	}
}

```

 符号函数
 本题要求实现符号函数sign(x)

 函数接口定义
 >  int sign(int x)

其中x是用户传入的整型参数。符号函数的定义为：若x大于0，sign(x) = 1；若x等于0，sign(x) = 0；否则，sign(x) = −1。

```C
#include<stdio.h>
int sign(int);
int main(void) {
	int x;
	scanf("%d", &x);
	printf("sing(%d) = %d \n", x, sign(x));
	return 0;
}

int sign(int n) { // 基本if判断
	if (n > 0) {
		return 1;
	}
	else if (n == 0)
	{
		return 0;
	}
	else
	{
		return -1;
	}
}

```

使用函数求奇数和

本题要求实现一个函数，计算N个整数中所有奇数的和，同时实现一个判断奇偶性的函数

函数接口定义

> int even( int n );
> int OddSum( int List[], int N );

```C
#include <stdio.h>

#define MAXN 10

int even(int);
int OddSum(int[], int);

int main(void) {
	int list[MAXN], n, i;
	scanf("%d", &n);
	printf("sum of( \n");
	for (i = 0; i < n; i++) {
		scanf("%d", &list[i]); {
			if (even(list[i]) == 0) {
				printf("%d ", list[i]);
			}
		}
	}
	printf(") = %d \n", OddSum(list, n));
	return 0;
}
int even(int n)
{
	if (n % 2 == 0) // 参数取余数判断是不是偶数
	{
		return 1;
	}
	else {
		return 0;
	}
}
int OddSum(int List[], int N)
{
	int i, sum = 0;
	for (i = 0; i<N; i++) // 循环整形数组 
	{
		if (List[i] % 2 != 0) // 判断当前循环是否为奇数
		{
			sum = sum + List[i]; // 奇数相加
		}
	}

	return sum;
}

```

使用函数计算两点间的距离

本题要求实现一个函数，对给定平面任意两点坐标(x1,y​1)和(x2,y2)，求这两点之间的距离。

函数接口定义

> double dist( double x1, double y1, double x2, double y2 );

```C
#include <stdio.h>
#include <math.h>

double dist(double x1, double y1, double x2, double y2);
int main(void) {
	double x1, y1, x2, y2;
	scanf("%lf %lf %lf %lf", &x1, &y1, &x2, &y2);
	printf("dist = %.2f \n", dist(x1, y1, x2, y2));
	return 0;
}
double dist(double x1, double y1, double x2, double y2) {
	return sqrt(pow((x1 - x2), 2) + pow((y1 - y2), 2)); // 这里百度了，基本原理是a² = b²+c² 直角三角形公式，使用math头文件提供的sqrt和pow两个函数进行计算。
}

```




 使用函数求素数和

 本题要求实现一个判断素数的简单函数、以及利用该函数计算给定区间内素数和的函数。
 素数就是只能被1和自身整除的正整数。注意：1不是素数，2是素数。
 函数接口定义

> int prime( int p );
> int PrimeSum( int m, int n );

其中函数prime当用户传入参数p为素数时返回1，否则返回0；函数PrimeSum返回区间[m, n]内所有素数的和。题目保证用户传入的参数m≤n。

```C
#include <stdio.h>
#include <math.h>

int prime(int p);
int primeSum(int m, int n);

int main(void) {
	int m, n, p;
	scanf("%d %d", &m, &n);
	printf("sum of( \n");
	for (p = m; p <= n; p++) {
		if (prime(p) != 0) {
			printf("%d ", p);
		}
	}
	printf(") = %d\n", primeSum(m, n));
	return 0;
}

// 定义prime 函数接受一个参数，判断这个参数是否为素数，是则返回1，不是则返回0
// 根据定义素数是指出了1和他本身之外不能被任整数整除
// 还有一种算法，但是看了一会没看懂。
int prime(int p){ // 这段代码通过
	int x = 0;
	if (p < 2) { // 小于2的都不是素数如果传入的数小于2 则直接返回0
		return 0;
	}
	for (int i = 2; i < p; i++) { // 通过for循环，从2开始对p进行取余，如果没有余数那么则不是素数。
		if (p%i == 0) {
			x++;
		}
	}
	if (x == 0) {
		return 1;
	}
	else {
		return 0;
	}
}

int primeSum(int m, int n) {
	int sum = 0;
	for (m; m < n; m++) {
		if (prime(m) != 0) {
			sum += m;
		}
	}
	return sum;
}

```

使用函数统计指定数字的个数
本题要求实现一个统计整数中指定数字的个数的简单函数。
函数接口定义

>int CountDigit( int number, int digit ); 

```C
#include <stdio.h>

int CountDigit(int number, int digit);

// number 是不超过整型的整数，digit为[0, 9]区间的整数，函数countdigit应该返回number重digit出现的次数

int main() {
	int number, digit;
	scanf("%d %d", &number, &digit);
	printf(" Number of digit %d in %d : %d \n", digit, number, CountDigit(number, digit));
	return 0;
}

int CountDigit(int number, int digit) {
	int sum = 0;
	int mun;
	if (number < 0) { // 如果传入的整数是负数则转换成正数
		number = -number;
	}
	if (number == 0 && digit == 0) { // 如果传入的数字和 匹配的数字都是0
		sum++;
	}
	while (number)
	{
		mun = number % 10; // 对10 取余数得到当前个位数的值
		if (mun == digit) {
			sum++;
		}
		number /= 10;// 除以10
	}
	return sum;
}
```

