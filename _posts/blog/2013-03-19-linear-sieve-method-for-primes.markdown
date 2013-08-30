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
		for(long i = 2 ; i <  MAXP ; i ++)  
		{  
			if(! isNotPrime[i])  
				prime[num_prime ++]=i;  
			for(long j = 0 ; j < num_prime && i * prime[j] <  MAXP ; j ++)  
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

为什么这里要break？先可以得出一个结论，此时的prime[j]为（i*prime[j]）的最小质因数。现设x=p1*a为合数，且p1为其最小的质因子，a为质数或合数，若为质数，则设a=1*p`（a`=1），否则设a=a`*p`，p`为质数（因为任意一个合数都可以表示成一个质数和另一个数的乘积）。则x=(p1*a`)*p`=p1*(a`*p`)，且可以知道p1*a`<=p`*a`,即合数（p1*a`）小于合数（p`*a`），且p1<=p`,故得出结论，即比一个合数数大的质数和该合数的乘积可用一个更大的合数和比其小的质数相乘得到。

在上面实现代码中，i在1~maxp间循环，在i=p1*a时，若满足 !(i % prime[j])而不break的话，有可能在后面遇到一个prime[k],使得某个数x`=p1`*a`*prime[k]，在之后i=(a`*prime[k])时，还会再和p1`相乘重新进行isNotPrime[i * prime[j]] = 1;  赋值，这样就造成了重复赋值，降低了效率，如果break了，则不会出现这样的情况。

考虑完上面的问题，还有个问题需要考虑，即省去的i*prime[k]在后来的过程中一定会再出现从而使得将isNotPrime[i * prime[k]]赋值为1么？答案是肯定的，因为上面提到了任意一个合数都可以表示成一个质数和另一个数的乘积，所以任意一个数总能表示成其最小质因子与另一个数相称的形式，因为我们枚举了所有可能的i，故“另一个数”的所有可能性我们都有考虑，所以，只要我们找到其最小质因子则就不会漏掉任意一个数。

再贴一个一般的筛法求素数，可以做一下比较~

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
