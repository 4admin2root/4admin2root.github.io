---
layout:     post
title:      转：LinuxKernelPanic报错解决思路
date:       2012-08-22
---
今天数据库 rac 节点1 又 kernel panic 了

摘录篇网文

原文：[http://blog.51osos.com/linux/linux-kernel-panic/](http://blog.51osos.com/linux/linux-kernel-panic/)

[GoFace](http://blog.51osos.com/author/goface/) 于 2011-8-9,17:22 

Linux虽然没有蓝屏现象，不过Kernel报错有时也会让人头疼。有时重启后正常，linux系统运行一段时间后又down了，总不能出现问题就reboot啊。我从网上搜集一下资料，整理了出来，希望大家能在评论与我交流您的看法与经验。

## 什么是Kernel Panic?

wiki:

A **kernel panic** is an action taken by an [operating system](http://en.wikipedia.org/wiki/Operating_system) upon detecting an internal [fatal error](http://en.wikipedia.org/wiki/Fatal_error) from which it cannot safely recover. The term is largely specific to [Unix](http://en.wikipedia.org/wiki/Unix) and [Unix-like](http://en.wikipedia.org/wiki/Unix-like) systems; for [Microsoft Windows](http://en.wikipedia.org/wiki/Microsoft_Windows)operating systems the equivalent term is [Bug check](http://en.wikipedia.org/wiki/Bug_check) (or, [colloquially](http://en.wikipedia.org/wiki/Colloquialism), [Blue Screen of Death](http://en.wikipedia.org/wiki/Blue_Screen_of_Death)).

The [kernel](http://en.wikipedia.org/wiki/Kernel_(computer_science)) routines that handle panics (in [AT&T](http://en.wikipedia.org/wiki/AT%26T)-derived and [BSD](http://en.wikipedia.org/wiki/BSD) Unix source code, a routine known as `panic()`) are generally designed to output an [error message](http://en.wikipedia.org/wiki/Error_message) to the [console](http://en.wikipedia.org/wiki/Computer_console), dump an image of kernel memory to disk for post-mortem[debugging](http://en.wikipedia.org/wiki/Debugging) and then either wait for the system to be manually rebooted, or initiate an automatic [reboot](http://en.wikipedia.org/wiki/Reboot_(computer)). The information provided is of highly technical nature and aims to assist a [system administrator](http://en.wikipedia.org/wiki/System_administrator) or [software developer](http://en.wikipedia.org/wiki/Software_developer) in diagnosing the problem.

Attempts by the operating system to read an invalid or non-permitted [memory address](http://en.wikipedia.org/wiki/Memory_address) are a common source of kernel panics. A panic may also occur as a result of a hardware failure or a bug in the operating system. In many cases, the operating system could continue operation after memory violations have occurred. However, the system is in an unstable state and rather than risking security breaches and data corruption, the operating system stops to prevent further damage and facilitate diagnosis of the error.

The kernel panic was introduced in an early version of [Unix](http://en.wikipedia.org/wiki/Unix) and demonstrated a major difference between the design philosophies of Unix and its predecessor [Multics](http://en.wikipedia.org/wiki/Multics). Multics developer [Tom van Vleck](http://en.wikipedia.org/wiki/Tom_van_Vleck) recalls a discussion of this change with Unix developer [Dennis Ritchie](http://en.wikipedia.org/wiki/Dennis_Ritchie):

I remarked to Dennis that easily half the code I was writing in Multics was error recovery code. He said, We left all that stuff out. If theres an error, we have this routine called panic, and when it is called, the machine crashes, and you holler down the hall, Hey, reboot it.<sup>[[1]](http://en.wikipedia.org/wiki/Kernel_panic#cite_note-0)</sup>

The original `panic()` function was essentially unchanged from Fifth Edition UNIX to the [VAX](http://en.wikipedia.org/wiki/VAX)-based UNIX 32V and output only an error message with no other information, then dropped the system into an endless idle loop. As the Unix[codebase](http://en.wikipedia.org/wiki/Codebase) was enhanced, the `panic()` function was also enhanced to dump various forms of debugging information to the console.

有两种主要类型kernel panic：

1.hard panic(也就是Aieee信息输出)2.soft panic (也就是Oops信息输出)

## 常见Linux Kernel Panic报错内容：

Kernel panic-not syncing fatal exception in interruptkernel panic  not syncing: Attempted to kill the idle task!kernel panic  not syncing: killing interrupt handler!Kernel Panic  not syncing：Attempted to kill init !

## 什么会导致Linux Kernel Panic?

只有加载到内核空间的驱动模块才能直接导致kernel panic，你可以在系统正常的情况下，使用lsmod查看当前系统加载了哪些模块。除此之外，内建在内核里的组件（比如memory map等）也能导致panic。

因为hard panic和soft panic本质上不同，因此我们分别讨论。

## **hard panic**

一般出现下面的情况，就认为是发生了kernel panic:

1. 机器彻底被锁定，不能使用
1. 数字键(Num Lock)，大写锁定键(Caps Lock)，滚动锁定键(Scroll Lock)不停闪烁。
1. 如果在终端下，应该可以看到内核dump出来的信息（包括一段Aieee信息或者Oops信息）
1. 和Windows蓝屏相似

**原因：**

对于hard panic而言，最大的可能性是驱动模块的中断处理(interrupt handler)导致的，一般是因为驱动模块在中断处理程序中访问一个空指针(null pointre)。一旦发生这种情况，驱动模块就无法处理新的中断请求，最终导致系统崩溃。

**信息收集**根据panic的状态不同，内核将记录所有在系统锁定之前的信息。因为kenrel panic是一种很严重的错误，不能确定系统能记录多少信息，下面是一些需要收集的关键信息，他们非常重要，因此尽可能收集全，当然如果系统启动的时候就kernel panic，那就无法只知道能收集到多少有用的信息了。

1. /var/log/messages: 幸运的时候，整个kernel panic栈跟踪信息都能记录在这里。
1. 应用程序/库 日志: 可能可以从这些日志信息里能看到发生panic之前发生了什么。
1. 其他发生panic之前的信息，或者知道如何重现panic那一刻的状态
1. 终端屏幕dump信息，一般OS被锁定后，复制，粘贴肯定是没戏了，因此这类信息，你可以需要借助数码相机或者原始的纸笔工具了。

如果kernel dump信息既没有在/var/log/message里，也没有在屏幕上，那么尝试下面的方法来获取（当然是在还没有死机的情况下）：

1. 如果在图形界面，切换到终端界面，dump信息是不会出现在图形界面的，甚至都不会在图形模式下的虚拟终端里。
<li>确保屏幕不黑屏，可以使用下面的几个方法：
<ul>
1. setterm -blank 0
1. setterm -powerdown 0
1. setvesablank off
</ul>
</li>
1. 从终端，拷贝屏幕信息（方法见上）

**完整栈跟踪信息的排查方法**

栈跟踪信息(stack trace)是排查kernel panic最重要的信息，该信息如果在/var/log/messages日志里当然最好，因为可以看到全部的信息，如果仅仅只是在屏幕上，那么最上面的信息可能因为滚屏消失了，只剩下栈跟踪信息的一部分。如果你有一个完整栈跟踪信息的话，那么就可能根据这些充分的信息来定位panic的根本原因。要确认是否有一个足够的栈跟踪信息，你只要查找包含EIP的一行，它显示了是什么函数和模块调用时导致panic。

**使用内核调试工具(kenrel debugger ,aka KDB)**

如果跟踪信息只有一部分且不足以用来定位问题的根本原因时，kernel debugger(KDB)就需要请出来了。KDB编译到内核里，panic发生时，他将内核引导到一个shell环境而不是锁定。这样，我们就可以收集一些与panic相关的信息了，这对我们定位问题的根本原因有很大的帮助。

使用KDB需要注意，内核必须是基本核心版本，比如是2.4.18，而不是2.4.18-5这样子的，因为KDB仅对基本核心有效。

## **soft panic**

**症状：**

1. 没有hard panic严重
1. 通常导致段错误(segmentation fault)
1. 可以看到一个oops信息，/var/log/messages里可以搜索到Oops
1. 机器稍微还能用（但是收集信息后，应该重启系统）

**原因：**

凡是非中断处理引发的模块崩溃都将导致soft panic。在这种情况下，驱动本身会崩溃，但是还不至于让系统出现致命性失败，因为它没有锁定中断处理例程。导致hard panic的原因同样对soft panic也有用（比如在运行时访问一个空指针)

**信息收集：**当soft panic发生时，内核将产生一个包含内核符号(kernel symbols)信息的dump数据，这个将记录在/var/log/messages里。为了开始排查故障，可以使用ksymoops工具来把内核符号信息转成有意义的数据。

为了生成ksymoops文件,需要：

- 从/var/log/messages里找到的堆栈跟踪文本信息保存为一个新文件。确保删除了时间戳(timestamp)，否则ksymoops会失败。
- 运行ksymoops程序（如果没有，请安装）
- 详细的ksymoops执行用法，可以参考ksymoops(8)手册。

## Kernel panic实例：

今天就遇到 一个客户机器内核报错：Kernel panic-not syncing fatal exception

重启后正常，几个小时后出现同样报错，系统down了，有时重启后可恢复有时重启后仍然报同样的错误。

我先来解释一下什么是**fatal exception**?

软件应用程序通过几个不同的代码层与操作系统及其他应用程序相联系。当异常（exception）在某个代码层发生时，为了查找所有异常处理的代码，各个代码层都会将这个异常发送给下一层，这样就能够处理这种异常。如果在所有层都没有这种异常处理的代码，致命异常（fatal exception）错误信息就会由操作系统显示出来。这个信息可能还包含一些关于该致命异常错误发生位置的秘密信息（比如在程序存储范围中的十六进制的位置）。这些额外的信息对用户而言没有什么价值，但是可以帮助技术支持人员或开发人员调试程序。

当致命异常（fatal exception）发生时，操作系统没有其他的求助方式只能关闭应用程序，并且在有些情况下是关闭操作系统本身。当使用一种特殊的应用程序时，如果反复出现致命异常错误的话，应将这个问题报告给软件供应商。 

而且此时键盘无任何反应，必然使用reset键硬重启。

方法：

#vi /etc/sysctl.conf  添加

kernel.panic = 20 #panic error中自动重启，等待timeout为20秒kernel.sysrq=1 #激活Magic SysRq  否则，键盘鼠标没有响应

按住 [ALT]+[SysRq]+[COMMAND], 这里SysRq是Print SCR键，而COMMAND按以下来解释！

配置一下以防万一。

很多网友安装linux出现Kernel panic-not syncing fatal exception in interrupt是由于网卡驱动原因。

解决方法：将选项Onboard Lan的选项Disabled,重启从光驱启动即可。

等安装完系统之后，再进入BIOS将Onboard Lan的选项给enable，下载相应的网卡驱动安装。

如出现以下报错：

init() r8168  

           

          ：Kernel panic: **Fatal exception**

r8168是网卡型号。

在BIOS中禁用网卡，从光驱启动安装系统。再从网上下载网卡驱动安装。

#tar  vjxf  r8168-8.014.00.tar.bz2

# make  clean  modules       (as root or with sudo)

安装好系统后reboot进入BIOS把网卡打开。

另有网友在Kernel panic出错信息中看到alc880，这是个声卡类型。尝试着将声卡关闭，重启系统，搞定。

安装linux系统遇到安装完成之后，无法启动系统出现Kernel panic-not syncing fatal exception。很多情况是由于板载声卡、网卡、或是cpu 超线程功能（Hyper-Threading ）引起的。这类问题的解决办法就是先查看错误代码中的信息，找到错误所指向的硬件，将其禁用。系统启动后，安装好相应的驱动，再启用该硬件即可。另外出现Kernel Panic  not syncing: attempted to kill init和Kernel Panic  not syncing: attempted to kill idle task有时把内存互相换下位置或重新插拔下可以解决问题。

本文至此结束，希望大家能在评论中交流您的看法和经历。

另 ：[http://blog.csdn.net/willand1981/article/details/5663356](http://blog.csdn.net/willand1981/article/details/5663356)
