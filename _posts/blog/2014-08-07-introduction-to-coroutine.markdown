---
layout: post
title: 协程
categories: blog
tags:
- 协程
- Python
- gevent
---

实习过程中的实习项目在实现web server时有接触到协程（Coroutine）的概念，也是第一次接触到，之前只是知道有线程、进程，而不知道还有协程的存在。

## 协程的概念

协程的概念最早是由Melvin Conway在1963年提出并实现，用于简化COBOL编译器的词法和句法分析器间的协作，当时他对协程的描述是“行为与主程序相似的子例程”。

[Wiki](http://en.wikipedia.org/wiki/Coroutine)上的定义是：
	
	Coroutines are computer program components that generalize subroutines to allow multiple entry points for suspending and resuming execution at certain locations. 

可以看到协程的一个特点就是它可以由多个挂起和恢复的点，也就是说可以在协程的内部的很多自定义的地方挂起当前协程或者从这个点恢复当前协程。这个看起来有点goto的意思了吧..

在wiki中举了一个生产者-消费者的例子：
	
	var q := new queue

	coroutine produce
    	loop
        	while q is not full
            	create some new items
            	add the items to q
        	yield to consume

    coroutine consume
    	loop
        	while q is not empty
            	remove some items from q
            	use the items
        	yield to produce

这个例子中，生产者生产直至队列满为止，然后利用了yield保留字跳转到consume的协程继续执行消费者的操作，直到消费者把队列里的元素都消耗完毕，然后调用yield在跳转到produce，但是这时候produce不是重头执行了，而是从刚才调用yield的地方开始执行。这就是协程的特点，它能从任意一个地方跳转，然后当跳转回来的时候还是从刚才跳转出去的地方执行，这个点也就是wiki上说的挂起和恢复的点，而这样的点在协程中可以由多个。

## 协程、线程和进程

进程有独立的堆和栈，进程间不会相互共享堆、栈，而且进程的调度是由操作系统完成的。

线程没有独立的堆和栈，它是存在于进程之中的，同一个进程中的线程共享同一个堆和栈，线程的调度也是由操作系统完成的。

对于上面两者的描述，可以简单的说，进程是操作系统中资源分配的最小单元，而线程是操作系统中CPU时间分配的最小单元。

协程，和线程很类似，区别在于协程的调度是由程序员完成的，也就是说协程间的跳转是由代码中明确设定的。

其实整个协程的执行和单线程中的函数执行是类似的，只不过在协程的执行过程中可以跳转到另外一个函数执行，而且这两个函数不是像线程间一样是并行执行的，而是一个"串行"执行的模式。在Python中，由于Python或者准确的说是CPython设计时候使用了[GIL](http://en.wikipedia.org/wiki/Global_Interpreter_Lock)，所以造成实际上Python的线程并不能在多核上同时运行。Python的多线程程序在运行的时候并不能充分地利用到多核资源，使得在运行的时候实际上要进行更多的线程切换。在这样的情况下，协程间的切换成本就低了很多，因为它只相当于你又开始执行另一个函数而已。所以在Python中，协程可以以更小的代价来实现和线程相同的效果，但是获得这样的优势的前提是协程间的调度要由程序提前设计好。

另外，在Python中协程的另一个应用就是生成器。关于生成器，可以看[这篇翻译自Stackoverflow的文章](http://pyzh.readthedocs.org/en/latest/the-python-yield-keyword-explained.html)，详细介绍了Python中的yield保留字和生成器的相关内容。

## 协程的实现

协程是一个应用层面的概念，所以操作系统和这个没有直接的关系，所以，可以利用某种语言来实现协程，这里用了C。

用C语言可以实现Python中yield的语义，代码如下：

	int function(void) {
		static int i, state = 0;
		switch (state) {
			case 0:
				goto LABEL0;
			case 1:
				goto LABEL1;
		}
		LABEL0:
		for (i = 0; i < 10; ++i) {
			state = 1;
			return i;
			LABEL1:;
		}
	}

在上面这段代码中，用了static静态变量的形式保存在每次函数执行的状态，同时利用switch语句和goto跳转实现了每次调用时对状态的判断和分支。

但是在这种实现下有小小问题，就是每当新添加一个return就要手动添加一个label，这显然不是很优雅，所以，可以利用__LINE__宏的方式来实现，同时也可以简化刚才的代码流程，把label和switch结合得更加紧密一些。

	int function(void) {
		static int i, state = 0;
		switch (state) {
			case 0:
			for (i  = 0; i < 10; ++i) {
				state = __LINE__ + 2;
				return i;
				case __LINE__:;
			}
		}
	}

在上面的代码中，显示把之前的label取消掉了，利用了swtich case的形式让代码变得更紧凑些，另外，更重要的是，利用了__LINE__宏使得每增加一个return都不用写硬代码，更加一般化。

接下来，既然这种结构可以做到一般化，那么不妨将它用宏的形式表现出来，使之更容易使用或者说更好看。

	#define BEGIN() static int state = 0; switch (state) { case 0:
	#define YIELD() do {state = __LINE__; return i; case __LINE__:;} while(0)
	#define END() }
	
	int function(void) {
		static int i;
	
		BEGIN();
		for (i = 0; i < 10; ++i) {
			YIELD();
		}
		END();
	}

这是用C的一个简单的协程实现。

## Python中的协程

(To be continued...)

## 参考文献

1. [一个“蝇量级” C 语言协程库](http://coolshell.cn/articles/10975.html)
2. [协程的C实现](http://www.hawkwithwind.net/blog/2011/02/18/%E5%8D%8F%E7%A8%8B%E7%9A%84c%E5%AE%9E%E7%8E%B0/)
3. [Coroutines via Enhanced Generators](http://legacy.python.org/dev/peps/pep-0342/)