title: Elf学习2
date: 2022-12
categories: [逆向]
tags: [elf,笔记]
---

文章摘要
<!--more-->


在系统范围禁用ASLR
   # echo "0" > /proc/sys/kernel/randomize_va_space
gcc test.c -o test -zexecstack -fno-stack-protector -g