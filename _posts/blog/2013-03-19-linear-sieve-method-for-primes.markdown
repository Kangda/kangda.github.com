---
layout: post
title: 线性筛法求素数
categories: blog
tags:
- 筛法
- 算法
- 素数
description: 线性筛法求素数，顾名思义，其时间复杂度为O(n)
---

线性筛法求素数，顾名思义，其时间复杂度为O(n)。

我是第一次接触线性筛法求素数，其中有些难理解的地方的确花了很多时间。

先放出代码：

	#include<iostream>
	using namespace std;
	const long MAXP = 200000;
	long prime[MAXP] = {0},num_prime = 0;
	int isNotPrime[MAXP] = {1, 1};
	
	int main()
	{
		for(long i = 2 ; i <  MAXP ; i++)
		{
			if(!isNotPrime[i])
				prime[num_prime++]=i;
			for(long j = 0 ; j < num_prime && i * prime[j] <  MAXP ; j++)
			{
				isNotPrime[i * prime[j]] = 1;
				if( !(i % prime[j]))
					break;
			}
		}
		return 0;
	}

难点在于

	if( !(i % prime[j])) break;

为什么这里要break？先可以作一个假设:

> 此时的prime[j]为（i\*prime[j]）的最小质因数。

现设x = p*a为合数，且p为其最小的质因子。a为质数或合数：若为质数，则设a = 1\*p\`（a\` = 1）；否则设a = a\`\*p\`，p\`为质数（因为任意一个合数都可以表示成一个质数和另一个数的乘积）。则，

> x = p\*a = p\*(a\`\*p\`) = p\`*(a\`\*p)

首先可以知道p\`也是x的一个质因子，同时a = a\`*p\`而且p是x**最小的**质因子，就是说**a > p**。

则可以知道

> a\`\*p <= a\`\*p\`

因为p是x最小的质因数，即p <= p\`，所以可得到上面。则得出结论，比一个质数p\`和一个合数的乘积可用**一个比p\`小的**质数p和一个**是p\`倍数的**合数a相乘得到，而且a\>p。

`!(i % prime[j])`为真的时候即等价于`i >= prime[j] 且 i是prime[j]的倍数`。在上面实现代码中，i在1~maxp间循环，在`i = p*a`时，进行`isNotPrime[p*a*prime[j]] = 1`的赋值，若此时满足`!(i % prime[j])`而不break的话，则会有可能在后面遇到一个更大的prime[k]，在之后`i = a*prime[k]`时，还会再和prime[j]（等于p）相乘重新进行`isNotPrime[a*prime[k]*p] = 1`的赋值，这样就造成了重复赋值，降低了效率，如果break了，则不会出现这样的情况。

考虑完上面的问题，还有个问题需要考虑，即省去的`i*prime[k]`在后来的过程中一定会再出现从而使得将`isNotPrime[i*prime[k]]`赋值为1么？答案是肯定的，因为任意一个合数都可以表示成一个质数和另一个数的乘积，所以任意一个数总能表示成其最小质因子与另一个数相称的形式，而且“另一个数”一定是大于这个最小质因子的，我们枚举了所有可能的i和**所有小于i的**质数，故所有可能性我们都有考虑，所以，不会漏掉任意一个数。

再贴一个一般的筛法求素数，可以做一下比较。

	void sieve()
	{
		memset(prime, 1, sizeof(prime));
		prime[0]=false;
		prime[1]=false;
		int m=31700;
		for (int i=2;  i<m;  i++)
			if (prime[i]) {
				primes[++cnt ]=i;
				for (int k=i*i; k<m; k+=i)
					prime[k]=false;
			}
		return;
	}

这个方法是很直观的方法，就是当前数的所有倍数都不可能是质数了（因为它有了出自己和1本身外的其他因子），那么就将它标记。但是这个方法里面没有上述所说的优化。
