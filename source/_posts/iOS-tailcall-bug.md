---
title: iOS App中与tail call相关的一个bug定位
date: 2018-12-28 21:35:19
tags:
---


今天遇到个崩溃，栈顶崩溃的地址是gcd里的一个内部函数，log中显示其直接调用者是我们的代码，但实际代码其实并未调用此函数。以下记录一下定位此问题过程中的一些收获。



拿到手的是一条崩溃log：

崩溃基本信息：
![tailcall_crash_header](/pics/tailcall_crash_header.png)

崩溃位置为thread 10，其调用栈如下：
![tailcall_crash_info](/pics/tailcall_crash_info.png)

<!-- more -->


**先从`SEGV_ACCERR at 0x70`开始。**

经查，栈顶的`dispatch_continuation_async`方法是由`dispatch_sync`调用的一个内部方法。通过查看`dispatch_continuation_async`的汇编代码，结合其开源实现，`SEGV_ACCERR at 0x70`应该是在调用`dispatch_continuation_async`时给queue参数传了0。

但与上图中Thread 10的崩溃栈相矛盾的是，`audioSourceDidStop`并未直接调用`dispatch_async`。

`audioSourceDidStop`的源码如下：
![tailcall_func_a](https://www.zhangjiezhi.com/pics/tailcall_func_a.png)

将崩溃详细信息中`lr`寄存器的地址经过`atos`进行符号解析后得到，崩溃当时的返回地址应为：`vadData2SpeexData`方法中的文件485行处：
```shell
$atos -arch arm64 -o xxx.dSYM/Contents/Resources/DWARF/XXX -l 0x10016c000 0x00000001003e5b80
-[SogouDefaultSpeechRecognizer vadData2SpeexData:status:beginSampleOffset:endSampleOffset:] (in SogouInput) (SogouDefaultSpeechRecognizer.m:485)
```

实际代码中`audioSourceDidStop`实际上是通过`postEndSignal`方法调用到了`vadData2SpeexData`，而此方法也的确调到了`dispatch_async`，其485行为`dispatch_async`的下一行。所以可以基本确定，崩溃时的现场的调用关系是：
> `audioSourceDidStop` -> `postEndSignal` -> `vadData2SpeexData`

其中`postEndSignal`的源码如下：
![tailcall_func_b](/pics/tailcall_func_b.png)

再经过进一步定位，崩溃原因应该是：
在`vadData2SpeexData`方法中，给dispatch_async传入的queue参数的确会被多个线程进行读写，在调用时的确可能为空：
![tailcall_func_c](/pics/tailcall_func_c.png)

所以基本可以确定修改方向为_speexQueue的多线程保护。


**但是为何崩溃栈中并未显示postEndSignal和vadData2SpeexData呢？**

首先怀疑是栈回溯出了问题，是不是栈被写坏了？但按理说如果写坏的话后面的frame也就找不到了，系统是如何在头部找不到的情况下找到更深层的frame pointer的呢？

带着这个问题查了一些文档，找到一个比较有趣的点：

apple的`arm64 calling convention`规范中有如下描述：
![tailcall_apple_arm64](/pics/tailcall_apple_arm64.png)

那么在刚才这个崩溃的现场有没有tail call呢？用hopper看了一下，在`postEndSignal`调用`vadData2SpeexData`时，的确是一个直接跳转，`postEndSignal`的汇编代码如下：
![tailcall_func_b_asm](/pics/tailcall_func_b_asm.jpg)

如红框中所示，在跳转到`objc_msgSend`（即调用`vadData2SpeexData`）前已经将栈和寄存器（包括`x29`和`x30`，即`fp`和`lr`）都恢复成调用`postEndSignal`之前的状态了，而且跳转用的是`b`指令。

这样对`vadData2SpeexData`的调用就像一个方法内的地址跳转一样，而且其执行环境为`audioSourceDidStop`中（类比将`vadData2SpeexData` inline到`audioSourceDidStop`中），即并没有给`vadData2SpeexData`分配frame record，而`postEndSignal`的执行已经结束。

崩溃时 fp寄存器存放的应该是`dispatch_continuation_async`的fp, 因此栈顶为此gcd方法；
其前驱fp为`audioSourceDidStop`的fp，找到其对应的pc为:`audioSourceDidStop`中的1048行所对应的地址。


** 至此，基本可以确定是一个多线程导致的崩溃，其头部的崩溃栈因为tail call的优化而被隐藏起来了。**