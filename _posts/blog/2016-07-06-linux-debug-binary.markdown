---
layout: post
title: Linux程序Debug
categories: blog
tags:
- code
- debug
description: Linux程序调试
---

最近在工作中，出现了一个BUG场景，但是因为没有足够的log，所以没办法判断BUG的具体位置，所以需要去debug相应的binary。在这个过程中接触到了一些debug的工具，所以就记录一下。

## gcore


gcore是一个用来生成core文件的工具，它其实是一个shell脚本，可以which到本地的gcore绝对路径然后vi它，可以看到具体的内容。gcore的主要实现事借助于gdb，是讲一个进程attach起来，然后利用gdb的gcore命令产生了core，然后再detach掉。

```shell
for pid in $*
do
        # `</dev/null' to avoid touching interactive terminal if it is
        # available but not accessible as GDB would get stopped on SIGTTIN.
        $binary_path/gdb </dev/null --nx --batch \
            -ex "set pagination off" -ex "set height 0" -ex "set width 0" \
            -ex "attach $pid" -ex "gcore $name.$pid" -ex detach -ex quit

        if [ -r $name.$pid ] ; then
            rc=0
        else
            echo "gcore: failed to create $name.$pid"
            rc=1
            break
        fi
done
```

gcore在实现中也是将进程attach起来，所以，从我的角度看，这样的操作在线上还是有风险的。而且，Linux没有提供对正在运行的进程生成core文件的系统接口，所以这样获取core文件应该也是唯一的选择了。

## gdb

gdb是调试的利器了，基本所有的调试操作最终都会在gdb中进行。可以对一个binary使用gdb，类似IDE的单步调试的功能，但是你得加breakpoint；另外，还可以加载core文件，但是同时也得有生成core文件的binary；最后，也可以对一个正在运行的程序进行调试，但是这个进程是要被attach的。

上述的三种方式分别对应了下面三种操作：

- gdb <binary>
- gdb <binary> <core-file>
- gdb <binary> <pid>

在gdb中常用的几个命令：

1. bt

查看当前运行的函数堆栈，注意，只是当前运行的线程的；

2. thread

查看当前进程的所有线程；

3. thread <tid>

切换到某个线程；

4. print <var>

输出某个变量的值，这个变量个表达式和C/C++风格的很类似，这里有个点比较有意思，就是在进行输出变量的类型转换的时候要在目标类型上加上单引号：

> p *('Test::A'*)ptr

另外，p命令可以查看当前所有的变量，只要是在当前环境中存在的变量。

5. list

显示当前环境对应的源码，当然，这个得需要在编译的时候添加相应的option，否则gdb也找不到源码；


## pstack

pstack可以打印出进程里所有线程当前的调用站，如果symbol信息还在的话，所有的symbol也会显示，当然这个还是依赖于编译时的设置。另外，这个操作也是隐式地进行了attach操作。

## objdump

objdump是一个查看二进制文件的工具，可以将二进制文件的信息和一些附加信息比较方便的展示出来。

TD..


