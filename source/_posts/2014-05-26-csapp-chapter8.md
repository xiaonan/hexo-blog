---
layout: post
title: "csapp chapter8:异常控制流"
description: "《深入理解计算机操作系统》英文名Computer Systems A programmer's Perspective的读书笔记"
category: "CSAPP"
tags: []
---
>程序运行的顺序是根据程序计数器（PC）的值而定的，PC从一个地址到另一个地址的过渡称为*控制转移(control transfer)*,这样的控制转移序列叫做处理器的*控制流(flow of control或 control flow)*。一般如果PC是按照顺序地址进行过渡的，控制流是一个平滑的序列，但是有些情况下PC是来回跳转的，这些突变称为*异常控制流(Exceptional Control Flow 简称EFC)*,EFC发生在计算机系统的各个层次。本章主要分析这种异常控制流。

##异常
异常(exception)就是控制流中的突变,用来响应处理器状态中的某些变化。
下面通过一个图来理解异常.
![](/assets/img/csapp/fig8.1.png)  
可以看到程序在执行到* \\(I_{curr}\\) *时发生了一个异常，导致程序从原来该执行* \\(I_{next}\\)* ，变为了执行异常处理, 在执行完异常出理由程序后有下面三种可能：

* 处理程序将控制返回给当前指令\\(I_{curr}\\), 即当事件发生时正在执行的命令。
* 处理程序将控制权返回给\\(I_{next}\\), 即如果没有发生异常将会执行的下一条指令。
* 处理程序终止被中断的程序。

在任何情况下，当处理器检测到有事件发生时，它就会通过一张*异常表*的跳转表,进行一个间接过程调用(异常)，到一个专门设计用来处理这类事件的操作系统子程序(异常处理程序(exception handler))。

##异常处理
系统为每个异常的类型都分配了一个唯一的非负整数的*异常号(exception number)*, 每个异常处理号都对应了一个异常处理程序，这些异常处理程序有些是处理器设计者分配的，有些事操作系统内核的设计者分配的，前者主要包括被零除，缺页，存储器访问违例、断点以及算术溢出，后者主要包括系统调用和来自外部I/O设备的信号。
在系统启动时，操作系统分配和初始化一张称为*异常表*的跳转表，如下：
![](/assets/img/csapp/fig8.2.png)  
这个图显示了如何利用根据异常号来找异常处理程序，并执行。  

**异常与过程调用的区别**：

* 异常调用时，在跳转处理器之前，处理器将返回地址压入栈中。返回地址是根据异常的类型而定的，可能是当前指令的地址，也可能是下一条指令的地址。过程调用一般入栈的是下一条指令的地址。
* 处理器把一些额外的处理器状态压到栈里，当处理程序返回时，重新开始被中断的程序会需要这些状态。
* 如果控制从一个用户程序转移到内核，那么所有的这些项目都被压入内核栈中，而不是压到用户栈中。
* 异常处理程序运行在内核模式下。这意味着它们对所有的系统资源具有完全的访问权限。

**异常的类别**:

* **中断**:当前指令执行完后，会发现中断引脚的电压高了，就从系统总线读取处理程序。这种异常一般都是由外部的设备引起的，跟处理程序之间没有必然的联系，所以是异步的。下面的都是程序运行过程中自己有意或无意的引起的，所以事同步的行为。
* **陷阱**:是一个有意的异常，是执行一条指令引起的，引起异常的指令一般都是系统调用函数（如fork, single, exit, waitpid等），这些函数调用使程序从用户模式切换到了内核模式。
* **故障**:故障是由程序运行中的错误造成的，它能够被捕获并被故障处理程序修正。这种故障如缺页异常等。
* **终止**:这种异常一般是指不会恢复的致命错误。通常是一些硬件错误。

|类别|原因|异步/同步|返回行为|
|---|---|---|---|
|中断|来自I/O设备的信号|异步|总是返回到下一条指令|
|陷阱|有意的异常|同步|总是返回到下一条指令|
|故障|潜在可恢复的错误|同步|可能返回当前指令|
|终止|不可恢复的错误|同步|不会返回|