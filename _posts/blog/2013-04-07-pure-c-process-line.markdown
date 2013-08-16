---
date: 2013-04-07 11:41:35+00:00
layout: post
title: 纯C语言处理行数据
categories: blog
tags:
- C++
---

C语言的输入输出虽然更底层，定义得更简单、灵活，但是在实现某些功能时有时会显得比较麻烦，这次就记一下关于C语言处理行数据的问题。

其实说行数据，也不过是先读取一整行的数据，然后再做处理，所以整个过程就分为读取和处理两部分。


##### 读取


读取上C的标准库里提供了函数来读取一行的数据，即

	/*Reads characters from the <em>standard input</em> (stdin) and stores them as a C string into <em>str</em> until a newline character or the <em>end-of-file</em> is reached.*/
	char * gets ( char * str );

	/*Reads characters from <em>stream</em> and stores them as a C string into <em>str</em> until (<em>num</em>-1) characters have been read or either a newline or the <em>end-of-file</em> is reached, whichever happens first.*/
	char * fgets ( char * str, int num, FILE * stream );

两者的区别就是前者是对标准输入的后者是可以针对流输入的。
两者除了输入来源不同之外，还有些其他的区别，比如，gets函数得到的结果不会包含最后的换行符（the ending newline character），但是fgets会；另外，gets不能限制读取的字符数，但是fgets可以。

两者得到的字符串的末尾都加了零字符（terminating null character）。

上面说的是利用C提供的函数读取一行，下面要说的方法就比较直接了，是通过读取一个一个字符来获取整行数据，主要利用的getchar函数

	/*Returns the next character from the standard input (<em>stdin</em>).*/
	int getchar ( void );

主要是通过判断读到的字符是不是'\n'来判断是不是读完了一行，这个方法和上面的效果是一样的，但是因为是自己实现的代码，所以在读取的过程中可以加一些运算操作来满足当前程序的一些其他要求。

关于这两种方法的代码实现，在最后会给出。


##### 处理


关于处理的话根据实现读取的方式不同也大致有两种方式，一种是在读取的过程中进行处理，一种是在读取完之后在进行处理。

显然第一种方式是针对逐字符读取的方式的，我们可以在读取的循环中通过判断当前读取到的字符的值来决定处理的方式，比如后面将要给出的例子里的根据空格来划分不同的词的一种处理方式，就要判断当前读到的是不是空格，如果是空格的话就要把已读到的存起来。

第二种方式的话，我们要借助另外一个C的输入输出函数

	/*Reads data from s and stores them according to parameter format into the locations given by the additional arguments, as if scanf was used, but reading from s instead of the standard input (stdin).*/
	int sscanf ( const char * s, const char * format, ...);

这个函数可以把我们已经读到的整行的数据当作输入源，然后从其中进行数据的格式化读取，格式化的方式与scanf是一样的。**要注意的是，这个函数和C++里的stringstream不太一样，主要是它不会像文件操作一样记录你当前读到哪个位置了，所以当第二次调用这个函数的时候，还是将从数据源的初始位置开始。**

主要的处理方式就是这些，下面贴出两段实践的代码

首先是逐字符读取的方式

	char c;
	
	l = 0;
	i = 0;
	while ((c = getchar()) && (c != '\n'))
	{
		if (c == ' ')
		{
			sen[l++][i] = '\0'; //给已分好的词填上结尾零字符
			i = 0;
			continue;
		}
		sen[l][i++] = c; //将读到的字符加入到当前的词字符串中
	}
	sen[l++][i] = '\0';

接下来是用gets方式读取的


	char line[MAX_SENTENCE_LENGTH + 1]; //为存储整行数据准备
	gets(line);
	l = 0;
	int offset = 0; //要手动存储当前读到的位置，便于读取下一个位置
	while ((strlen(line) > offset) && EOF != sscanf(line + offset, "%s", sen[l++]))
	{
		offset += strlen(sen[l-1]) + 1; //更新读取位置
	}
