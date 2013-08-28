---
date: 2013-03-28 16:18:27+00:00
layout: post
title: CSAPP Notes(3.1-3.6)
categories: blog
tags:
- assembly language
- CSAPP
- note
description: CSAPP阅读笔记
---

一点点来，前面两章已经看过了（除了浮点数那部分），所以，从第三章开始


### Chapter 3 Machine-Level Representation of Programs


3.1和3.2讲的是一些历史和介绍性的东西，而且已经看过了，有些还是值得记得，**以后要补上**，就先跳过了..


#### 3.3 Data Formats


主要是一张表，先把这张表放在这


[![size of c data types in IA32](http://kang-da.tk/images/post/size-of-c-data-types-in-IA32.png)](http://172.18.218.161/blog/wp-content/uploads/2013/03/size-of-c-data-types-in-IA32.png)




这里要注意的是在汇编语言里不同的操作数的大小的表示方法和它们的实际大小。简单说来，就是b、w、l、s、t，其中bwl在之前学汇编的时候倒是了解过，但是浮点数的表示就没怎么接触过了。





#### 3.4 Accessing Information


这节主要写的是关于mov的一些操作，即数据的访问。首先，介绍了下IA32下的寄存器，32位寄存器有%eax、%ecx、%edx、%ebx、%esi、%edi、%esp、%ebp；16位寄存器有%ax、%cx、%dx、%bx、%si、%di、%sp、%bp；8位寄存器有%ah、%al、%ch、%cl、%dh、%dl、%bh、%bl。这里面%esp、%sp是栈指针（Stack pointer），%ebp、%bp是帧指针（Frame pointer）。

下面就是这节的一个小亮点，就是汇编下关于操作数的表示，也是一张表

[![operand forms](http://kang-da.tk/images/post/operand-forms.png)](http://kang-da.tk/images/post/operand-forms.png)

其中，要注意的是立即数（Immediate）的部分，若想要表示立即数本身的值的话要在前面加上**$符号**，而直接写一个数字上去，会被当作是内存地址..**还有s只能取1,2,4,8（还没搞清楚为什么..）**

另外，基本括号里都是寄存器的操作，上面写得有点冗余，细想下来，涉及寄存器的表示有下面三种：



	
  1. Imm(E)，表示M[Imm+R[E]]；

	
  2. Imm(E1,E2)，表示M[Imm+R[E1]+R[E2]];

	
  3. Imm(E1,E2,s)，表示M[Imm+R[E1]+R[E2]*s].


以上的Imm都是可以省略的，另外，三参数的E1也是可以省略的。其实，再想想，主要就是最下面那种三参数的形式，其他的都是他派生演变的嘛。

AT&T汇编和Intel汇编的一个小区别让我很有印象，AT&T的偏移量都是在基地址前的，Intel应该是在之后的。

接下来就是要介绍mov指令了，关于mov指令，同样有一张表来介绍

[![data movement instructions](http://kang-da.tk/images/post/data-movement-instructions.png)](http://kang-da.tk/images/post/data-movement-instructions.png)

这部分总结下来分为三个部分：操作数大小相同，操作数大小不同，栈操作。

当操作数大小相同时，mov后面加上相应大小的表示字母（b/w/l）来区分针对不同大小的指令；操作数大小不同时，只能从小的类型转向较大的类型，而这其中又有了两种不同的高位填充，自然是符号填充（s）和零填充（z），而其后用操作数的大小字母来区分指令；栈操作指令相对简单，只有double word（l）一种，而操作也是常见的push和pop。

下面，要说的这个我觉得挺有用，就是mov指令两个操作数所有可能的来源（Immediate/Register/Memory）：

[![five combinations](http://kang-da.tk/images/post/five-combinations.png)](http://kang-da.tk/images/post/five-combinations.png)

总结下：



	
  * Imm只能做源操作数（Source Operand），目标操作数（Destination Operand）可以是寄存器和内存

	
  * Reg既可以做源操作数又可以做目标操作数

	
  * Mem做源操作数时，目标操作数只能是寄存器；Mem做目标操作数，源操作数可以是任意


有要注意的一点是，不存在(Mem,Mem)的情况，就是说想要把一个内存数据转存到另一个内存地址，要分两条指令。


#### 3.5 Arithmetic and Logical Operations


这节主要是介绍了几个算数运算的指令，相对上一节讲的一些generalized的内容来说，这节是比较detailed的。主要是两个表，先来说第一个表

[![interger arithmetic operations](http://kang-da.tk/images/post/interger-arithmetic-operations.png)](http://kang-da.tk/images/post/interger-arithmetic-operations.png)

这张表上划分了四个部分，第一个部分是一个取地址的指令；第二部分是单操作数指令（unary operation）；第三部分是双操作数指令（binary operation）；最后是移位操作。

首先，第一部分中这条指令的表达式就说明了它的作用，是用来取内存单元的地址的，把取到的地址放在目标操作数中，且目标操作数必须是寄存去。但同时，也有一个附加的小作用，比如指令

	leal 7(%eax, %edx, 4)

这条指令的实际效果是，假设寄存器%edx的值是x，则最后会将4x+x+7存入寄存去%eax，即可以利用这样的方式来做一些简单的运算。

再看第二部分，这部分相对简单些，主要是实现自加、自减、按值取反、按位取反。然后，根据操作数的大小，可以在指令后面添上b、w、l之类的字母来标识针对的操作数大小。

第三部分的指令和第二部分的差不多，主要是一些基本操作。

第四部分是移位操作，这里要注意几点：1.移动的位数只能是0～31,因为只有k值（移动的位数）的后5位会被考虑，至于为什么，估计就是这样涉及的吧；2. 移动位数可以是immediate也可以是寄存器%cl，而且如果是寄存器的话，只允许%cl；3.SAL和SHL都是左移操作，而且效果相同，都是在右边添0,但是SAR和SHR就不一样了，前者是算术移位，即以符号为为填充，后者是逻辑移位，以0为填充。同样，指令后添上b、w、l可以标识操作数的大小。

接下来讨论的是一些特别的算术操作（Special Arithmetic Operations），上这一节的第二个表格

[![special arithmetic operations](http://kang-da.tk/images/post/special-arithmetic-operations.png)](http://kang-da.tk/images/post/special-arithmetic-operations.png)

有几点要注意先：1.之前那张表的IMUL指令也是有符号数的操作

这里就是乘法和除法的处理了，之前的那张表里已经有了指令IMUL，但是那条指令的结果和操作数都是同样大小的，这是不合理的，因为，两个32位的操作数相乘得到的结果最大可能是一个64位的数。

看指令的表达式可以知道，这里的指令，除了cltd，都是默认参与运算的一个操作数是存在与寄存器中的。乘法的一个乘数是默认在%eax中，而除法的被除数则默认在由%edx和%eax连接组成的64位数中的。在结果的存放方面，乘法是把得出的结果存在%edx和%eax组成的64位中，除法则是产生两个结果，一个商，一个余数，其中，商是存在%eax中，余数存在%edx中。

imull和mull的区别在于一个数对有符号数的操作，一个是对无符号数的操作，idivl和divl的区别类似。

再来，最后就是cltd指令，从指令的效果上来看，应该是来准备被除数用的，将一个32位的数扩充到%edx和%eax组成的64位数。

运算的操作指令就基本是这样了。


#### 3.6 Control


这一节分为两个部分吧，一个是关于控制指令的介绍，一部分是一些高级语言里的控制语句和汇编指令的对应。

这里主要写下关于前一部分的。

首先，控制总是需要判断条件的，这些条件就是一些标志寄存器，它们存着当前运算的一些状态信息，常用的标志寄存器有：

[![flags](http://kang-da.tk/images/post/flags.png)](http://kang-da.tk/images/post/flags.png)

其中，CF是进位标志，ZF是零标志，SF是符号标志，OF是溢出标志。OF的溢出当然是针对有符号数来说的，也就是这本书里所说的two's-complement数。

关于标志寄存器的要注意：INC和DEC指令可以影响OF和ZF但是对CF没有影响。

但是，光是这些标志寄存器还是不够的，因为我们要有相应的运算来填充标志寄存器才能让我们获得状态信息，所以，要引入一些测试或者说是判断指令：

[![Comparison and test instruction](http://kang-da.tk/images/post/Comparison-and-test-instruction1.png)](http://kang-da.tk/images/post/Comparison-and-test-instruction1.png)

上面的指令执行后，然后运算过程中会根据指令的含义填充各个标志寄存器，说的直白些就是专门用来写标志寄存器的。

这里的指令分为两类，其实如果用一个牵强的方法来分类的话，一个是算术判断指令，一个是逻辑判断指令。其中，CMP就是判断两个操作数的大小了，与SUB指令基本是一样的；而TEST是将两个操作时相与从而获得想要的信息，与AND指令类似，但是有一点不同，AND会改变目标操作数的值，使其为两操作数与运算后的结果，但是TEST不会改变任何操作数的值，只是做与运算。

有了存有运算状态的标志寄存器以后，我们就要使用其中的状态信息，获取这些状态信息也有一组指令：

[![the set instruction](http://kang-da.tk/images/post/the-set-instruction.png)](http://kang-da.tk/images/post/the-set-instruction.png)

一般，标志寄存器的值的使用有三个方式：1,根据这些状态信息来将某一个字节置1或者0；2,根据状态信息决定跳转到的程序的某个部分；3,根据状态信息选择性传输数据。

首先说到的是第一种方式。

SET指令只有一个操作数，而且这个操作数的大小必须是**1字节**，不过，操作数可以是寄存器或者是内存地址。SET指令只关心标志寄存器的值，而不关心产生这些标志位信息的运算的操作数的大小。

由上表可以看到，除了前两部分是简单的把寄存器的值赋给操作数外，其余两部分都是关于符号的。其中，与有符号判断有关的用great(g)、less(l)、equal(e)来标识，而无符号判断用above(a)、below(b)、equal(e)来标识。

在指令设计时，有符号的判断要考虑进溢出(overflow)的情况，这也就是为什么有符号判断部分有OF的参与。

再来说到的是第二种方式，即跳转到程序的某个部分。

先上指令

[![the jump instructions](http://kang-da.tk/images/post/the-jump-instructions.png)](http://kang-da.tk/images/post/the-jump-instructions.png)

其实，JUMP指令集与SET指令集挺相似的，只不过，前者的效果是跳转，后者的效果是置位。既然这样，那我们主要来看一些前两个指令，即无条件跳转指令JMP。

JMP可以直接跳转到某个标签（Label）这个当然不用多说的，主要看下另一条。这条指令是跳转到某个具体指定的地址，这个地址可以存在寄存器，也可以存在内存，于是有个两种表示方式：jmp *%eax, jmp *(%eax)。其中前者的目标跳转地址是寄存器里的值，而后者的目标地址则是寄存器里存的值对应的内存地址下的值。

汇编器（assembler）或者链接器（linker）对跳转目标可能有不同表示，有的汇编器或者链接器会用相对地址，而有的则会用绝对地址。有的是PC相关的（PC-relative），有的则不是。这根剧不同的汇编器或链接器的具体设计而不同。

还有一点要注意，进行相对地址的跳转时，是以当前指令的下一条指令为相对的基准的，这是一个惯例，是由于早期的CPU会在执行当前指令时就将program counter（PC）指向下一条指令。

至此，汇编的指令集基本就介绍完了。


###### Loops


在循环中，跳转指令是一定要出现的。C中提供的循环有三种：do-while、while、for。其中最基本的是do-while循环，大多数的编译器都会通过do-while循环的代码来实现其他两种循环。

首先，来看do-while循环。一般形式是


	do
		_body-statement_
	while (_test-expr_);


把上面的基本结构翻译成goto语句的版本就是下面：


	loop:
	_body-statement_
	t = _test-expr_;
	if (t)
		goto loop;


上面这个结构和汇编语言就比较相近了，所以比较容易翻译成汇编，主要是用JUMP指令。

接下来，while循环和do-while相比，多了一个初始条件的判断，一般while循环的结构是


while (_test-expr_)
	_body-statement_

它对应的do-while循环的表达是


	if (!_test-expr_)
		goto done;

	do
		_body-statement_
	while (_test-expr_);
	done:


然后可以根据do-while的翻译，将其翻译成汇编代码。

最后就是for循环了，for循环和while循环相比的话多了一个初始化操作，结构是


	for (_init-expr_; _test-expr_; _update-expr_)
		_body-statement_


对应的while循环是

	_init-expr;_
	while (_test-expr_) {
		_body-statement_
		_update-expr;_
	}


对应的do-while的结构是


	_init-expr;_
	if (!_test-expr_)
		goto done;
	do {
		_body-statement_
		_update-expr;_
	} while (_test-expr_);
	done:

###### Conditional Move Instructions


（TO BE CONTINUED..）
