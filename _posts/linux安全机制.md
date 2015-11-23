title: linux安全机制
categories: [linux]
date: 1912-12
tags: [笔记]
---
ubuntu下可用的安全机制矩阵：https://wiki.ubuntu.com/Security/Features
看了一会，简单罗列下，不知道理解对没：
No Open Ports：系统默认安装完后，没有任何远程可访问的监听端口。
1，127.x.x.x的端口不算，其属于本机回路测试IP地址，因此远程无法访问。
2，特例：DHCP或mDNS。
3，系统管理员安装相关服务程序后，比如apache，可以指定对应的监听端口。

Password hashing：系统密码存放在/etc/shadow，从Ubuntu 8.10开始，使用SHA-512替代之前的MD5做哈希。

SYN cookies：抵御SYN-flood DDOS工具。

Filesystem Capabilities：现代文件系统支持xattrs扩展属性，减少setuid风险。
http://plaintext.blog.edu.cn/home.php?mod=space&uid=1557851&do=blog&id=382146

Configurable Firewall：ufw防火墙，iptables的前端界面。

Cloud PRNG seed：用来协助产生伪随机码，比较适合云环境。

PR_SET_SECCOMP：使一个进程进入到一种“安全”运行模式，该模式下的进程只能调用4种系统调用（system calls），即read(), write(), exit()和sigreturn()，否则进程便会被终止。
http://note.sdo.com/u/634687868481358385/NoteContent/M5cEN~kkf9BFnM4og00239

AppArmor\SELinux\SMACK：强制访问控制机制，指定应用程序可以执行或不可以执行的操作。

Encrypted LVM：从Ubuntu 12.10后，Ubuntu可安装到一整块加密的lvm上，因此所有分区都在一个逻辑分卷里，包括交换分区，并且都是加密的。在安装过程中选择：guided – use tntire disk and set up encrypted LVM。

eCryptfs：eCryptfs 是在 Linux 内核 2.6.19 版本中引入的一个功能强大的企业级加密文件系统，堆叠在其它文件系统之上（如 Ext2, Ext3, ReiserFS, JFS 等），为应用程序提供透明、动态、高效和安全的加密功能。
Ecryptfs是一个强大的本地加密软件。特点如下：
支持文件粒度：加密到每一个文件。你可以单独取出一个文件进行解密。而不是像其它软件那样，制作一个大大的文件包。
系统内核支持：解密文件夹挂载点（在哪查看解密文件夹），对用户和系统是透明的（与平常文件夹没有区别）。可以对复杂软件进行加密支持，比如sql数据库文件夹、web文件夹、聊天记录（对软件没有影响，也不需设置）。
多种加密方式：与普通软件不同，它提供了不同强度、方式的加密。
http://www.ibm.com/developerworks/cn/linux/l-cn-ecryptfs/
http://wiki.ubuntu.org.cn/Ecryptfs

Stack Protector/Heap Protector/Pointer Obfuscation：开发人员使用

Address Space Layout Randomisation (ASLR)：是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，通过增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。据研究表明ASLR可以有效的降低缓冲区溢出攻击的成功率。
cat /proc/sys/kernel/randomize_va_space
0 – 表示关闭进程地址空间随机化。
1 – 表示将mmap的基址，stack和vdso页面随机化。
2 – 表示在1的基础上增加栈（heap）的随机化。

Built as PIE：位置独立的可执行区域（position-independent executables）。这样使得在利用缓冲溢出和移动操作系统中存在的其他内存崩溃缺陷时采用面向返回的编程（return-oriented programming）方法变得难得多。
gcc -fPIE -pie …

Built with Fortify Source：强化安全编译，通过”-D_FORTIFY_SOURCE=2″或-O1及以上优化，即可开启该项安全保护。
1，扩展安全函数：自动替换使用sprintf、strcpy等的n版函数，也就是snprintf、strncpy等，避免缓冲区溢出。
2，当格式字符串是一个可写的内存段适合，阻止%n格式串，攻击。
3，检查一些重要函数（比如system，write，open等）的返回状态码和参数。
4，创建新文件时需要指定文件mask掩码。

Built with RELRO、Built with BIND_NOW：设置符号重定向表格为只读或在程序启动时就解析并绑定所有动态符号，从而减少对GOT（Global Offset Table）攻击。
RELRO: RELocation Read-Only：http://isisblogs.poly.edu/2011/06/01/relro-relocation-read-only/

Non-Executable Memory：设置某些内存区域（heap、stack等）为不可执行（Non-eXecute (NX) or eXecute-Disable (XD)）。需要BIOS开启（从Ubuntu 11.04开始，Ubuntu忽略BIOS的设置，即不管BIOS是否开启，只要CPU支持，那么Ubuntu就会使用它）。需要开启PAE（64bit或32bit-server或32bit-generic-pae均以开启）。

/proc/$pid/maps protection：限制maps文件只能被本身进程或进程所有者修改，其他均为只读。

其它：….
