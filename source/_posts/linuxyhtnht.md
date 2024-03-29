---
title: Linux的用户态和内核态
date: 2017-07-16 15:18:39
tags:
 - Linux
 - 用户态
 - 内核态
categories:
 - Linux/Unix
---

**内核态**:：CPU 可以访问内存所有数据，包括外围设备：硬盘、网卡。CPU 也可以将自己从一个程序切换到另一个程序。

**用户态**：只能受限的访问内存，且不允许访问外围设备。占用 CPU 的能力被剥夺，CPU 资源可以被其他程序获取。

### Unix/Linux的体系架构

![2017-7-16 152145](/images/2017-7-16 152145.png)

如上图所示，从宏观上来看，Linux 操作系统的体系架构分为用户态和内核态（或者用户空间和内核）。

**内核从本质上看是一种软件—控制计算机的硬件资源，并提供上层应用程序运行的环境**。

用户态即上层应用程序的活动空间，应用程序的执行必须依托于内核提供的资源，包括 CPU 资源、存储资源、I/O 资源等。为了使上层应用能够访问到这些资源，内核必须为上层应用提供访问的接口：即系统调用。

**系统调用是操作系统的最小功能单位**，这些系统调用根据不同的应用场景可以进行扩展和裁剪，现在各种版本的Unix实现都提供了不同数量的系统调用，如 Linux 的不同版本提供了 240-260 个系统调用，FreeBSD 大约提供了 320 个（reference：UNIX环境高级编程）。我们可以把系统调用看成是一种不能再化简的操作（类似于原子操作，但是不同概念），有人把它比作一个汉字的一个“笔画”，而一个“汉字”就代表一个上层应用。因此，有时候如果要实现一个完整的汉字（给某个变量分配内存空间），就必须调用很多的系统调用。如果从实现者（程序员）的角度来看，这势必会加重程序员的负担，良好的程序设计方法是：重视上层的业务逻辑操作，而尽可能避免底层复杂的实现细节。库函数正是为了将程序员从复杂的细节中解脱出来而提出的一种有效方法。它实现对系统调用的封装，将简单的业务逻辑接口呈现给用户，方便用户调用，从这个角度上看，库函数就像是组成汉字的“偏旁”。这样的一种组成方式极大增强了程序设计的灵活性，对于简单的操作，我们可以直接调用系统调用来访问资源，如“人”，对于复杂操作，我们借助于库函数来实现，如“仁”。

Shell 是一个特殊的应用程序，俗称命令行，本质上是一个命令解释器，它下通系统调用，上通各种应用，通常充当着一种“胶水”的角色，来连接各个小功能程序，让不同程序能够以一个清晰的接口协同工作，从而增强各个程序的功能。同时，Shell 是可编程的，它可以执行符合 Shell 语法的文本，这样的文本称为Shell 脚本，通常短短的几行 Shell 脚本就可以实现一个非常大的功能，原因就是这些 Shell 语句通常都对系统调用做了一层封装。为了方便用户和系统交互，一般，一个 Shell对应一个终端，终端是一个硬件设备，呈现给用户的是一个图形化窗口。我们可以通过这个窗口输入或者输出文本。这个文本直接传递给shell进行分析解释，然后执行。

用户态的应用程序可以通过三种方式来访问内核态的资源：

- 系统调用；
- 库函数；
- Shell 脚本。

下图是对上图的一个细分结构，从这个图上可以更进一步对内核所做的事有一个“全景式”的印象。

主要表现为：

- 向下控制硬件资源；
- 向内管理操作系统资源：包括进程的调度和管理、内存的管理、文件系统的管理、设备驱动程序的管理以及网络资源的管理；
- 向上则向应用程序提供系统调用的接口。

从整体上来看，整个操作系统分为两层：用户态和内核态，这种分层的架构极大地提高了资源管理的可扩展性和灵活性，而且方便用户对资源的调用和集中式的管理，带来一定的安全性。

![2017-7-16 152540](/images/2017-7-16 152540.jpg)

### 用户态和内核态的切换

因为操作系统的资源是有限的，如果访问资源的操作过多，必然会消耗过多的资源，而且如果不对这些操作加以区分，很可能造成资源访问的冲突。所以，为了减少有限资源的访问和使用冲突，Unix/Linux 的设计哲学之一就是：对不同的操作赋予不同的执行等级，就是所谓特权的概念。简单说就是有多大能力做多大的事，与系统相关的一些特别关键的操作必须由最高特权的程序来完成。

Intel 的 X86 架构的 CPU 提供了 0 到 3 四个特权级，数字越小，特权越高，Linux 操作系统中主要采用了 0 和 3 两个特权级，分别对应的就是内核态和用户态。运行于用户态的进程可以执行的操作和访问的资源都会受到极大的限制，而运行在内核态的进程则可以执行任何操作并且在资源的使用上没有限制。很多程序开始时运行于用户态，但在执行的过程中，一些操作需要在内核权限下才能执行，这就涉及到一个从用户态切换到内核态的过程。比如 C 函数库中的内存分配函数 malloc()，它具体是使用 sbrk() 系统调用来分配内存，当 malloc 调用 sbrk() 的时候就涉及一次从用户态到内核态的切换，类似的函数还有 printf()，调用的是 wirte() 系统调用来输出字符串，等等。

用户态到内核态的切换方式：

- 系统调用；
- 异常事件： 当 CPU 正在执行运行在用户态的程序时，突然发生某些预先不可知的异常事件，这个时候就会触发从当前用户态执行的进程转向内核态执行相关的异常事件，典型的如缺页异常。
- 外围设备的中断：当外围设备完成用户的请求操作后，会像 CPU 发出中断信号，此时，CPU 就会暂停执行下一条即将要执行的指令，转而去执行中断信号对应的处理程序，如果先前执行的指令是在用户态下，则自然就发生从用户态到内核态的转换。

**注意**：系统调用的本质其实也是中断，相对于外围设备的硬中断，这种中断称为软中断，这是操作系统为用户特别开放的一种中断，如 Linux int 80h 中断。所以，从触发方式和效果上来看，这三种切换方式是完全一样的，都相当于是执行了一个中断响应的过程。但是从触发的对象来看，系统调用是进程主动请求切换的，而异常和硬中断则是被动的。



Read More:

> [内核态(Kernel Mode)与用户态(User Mode)](http://www.cnblogs.com/zemliu/p/3695503.html)
>
> [Linux探秘之用户态与内核态](http://www.cnblogs.com/bakari/p/5520860.html)

