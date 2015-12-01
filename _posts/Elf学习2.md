title: Elf学习2
date: 2022-12
categories: [逆向]
tags: [elf,笔记]
---
info b
save breakpoint fig8.3.bp
gdb fig8.3 -x fig8.3.bp
<!--more-->


在系统范围禁用ASLR
   # echo "0" > /proc/sys/kernel/randomize_va_space
gcc test.c -o test -zexecstack -fno-stack-protector -g

如果你想跟踪子进程进行调试，可以使用set follow-fork-mode mode来设置fork跟随模式。参数可以是以下的一种：
    parent
        gdb只跟踪父进程，不跟踪子进程，这是默认的模式。
    child
        gdb在子进程产生以后只跟踪子进程，放弃对父进程的跟踪。
进入gdb以后，我们可以使用show follow-fork-mode来查看目前的跟踪模式

同时调试父进程和子进程，以上的方法就不能满足了。Linux提供了set detach-on-fork mode命令来供我们使用。其使用的mode可以是以下的一种：
    on
        只调试父进程或子进程的其中一个(根据follow-fork-mode来决定)，这是默认的模式。
    off
        父子进程都在gdb的控制之下，其中一个进程正常调试(根据follow-fork-mode来决定)
    另一个进程会被设置为暂停状态。

同样，show detach-on-fork显示了目前是的detach-on-fork模式，如上图。

    以上是调试fork产生子进程的情况，但是如果子进程使用exec系统函数而装载了新程序执行呢？——我们使用set follow-exec-mode mode提供的模式来跟踪这个exec装载的程序。mode可以是以下的一种：

    new 当发生exec的时候，如果这个选项是new，则新建一个inferior给执行起来的子进程，而父进程的inferior仍然保留，当前保留的inferior的程序状态是没有执行。

    same 当发生exec的时候，如果这个选项是same(默认值)，因为父进程已经退出，所以自动在执行exec的inferior上控制子进程。
