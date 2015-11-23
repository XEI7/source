title: Elf学习
categories: [逆向]
date: 1912-12
tags: [elf,笔记]
---
文章摘要
<!--more-->
 链接器对编译生成的目标文件进行链接时，
1.首先进行符号解析，找出外部符号在哪定义。如果外部符号在一个静态库中定义，则直接将对应的定义代码复制到最终生成的目标文件中。
2.接着链接器进行符号重定位。
 编译器在生成目标文件时，通常使用从零开始的相对地址，而在链接过程中，链接器从一个指定的地址开始，根据输入目标文件的顺序，以段（segment）为单位将它们拼装起来。其中每个段可以包括很多个节（section）。除了目标文件的拼装，重定位过程中还完成了下面两个任务：一是生成最终的符号表，二是对代码段(.text)中的某些位置进行修改，要修改的位置由编译器生成的重定位表指出。
 链接过程中还会生成两个表：got表和plt表。

got表中每一项都是本运行模块要引用的全局变量或函数的地址，可以用got表来间接引用全局变量。
函数也可以把got表的首地址作为一个基准，用相对该基准的偏移量来(直接)引用静态函数。
由于动态链接器（ld-linux.so）不会把运行模块(*.so)加载到固定地址，在不同进程的地址空间中各运行模块的绝对地址、相对地址都不同。
这种不同反映到got表上，就是每个进程的每个运行模块都有独立的got表，所以进程间不能共享got表(内容)。

plt表中每一项都是一小段汇编代码，对应于本运行模块要引用的每一个全局函数。当链接器发现某个符号引用是位于其它共享目标文件(动态连接库*.so)中的一个全局函数时，就在文件的plt表里创建一个项目，以便将来重定位。

  3.4.1 节区头部表格

    表 5 节区头部表格中的特殊下标

    SHN_COMMON OXFFF2 此节区定义的符号是公共符号。如 FORTRAN 中 COMMON 或者未分配的 C 外部变量。

    

    （《linux内核完全剖析》P42

    汇编器1次只处理1个源程序，如果变量不在本文件里定义，汇编器就留个记号，交给连接器处理。连接器对未定义的符号进行特殊处理。1。如果未定义符号里数

    值是0，则表示该符号在本汇编源程序里没有定义，连接器会尝试到其他连接的文件里确定它的数值。比如：在一个源程序里使用了一个符号，但没有对符号进行定

    义，汇编器处理汇编源程序时就会产生这种未定义符号，其值等于0。 2。如果未定义符号的数值不为0，那么该符号的值就表示是 .comm

    公共声明（as汇编器汇编命令：.comm symbol,length

    该命令在bss节区里，声明一个命名的公共存储区域）的需要保留的公共存储空间的字节长度。符号指向该存储空间的第1个地址处。汇编器1个1个的处理源文

    件，产出多个.o文件，多个.o文件里可能有多个同名公共符号。连接器在连接过程中，会把多个目标文件中的同名的公共符号合并。如果连接器没有找到1个符

    号的定义（而且此符号的值不为0），而是找到1个或多个公共符号，那么连接器就会分配长度为length字节的未初始化内存。length必须是一个绝对

    值表达式，如果连接器找到多个长度不同但同名的公共符号，连接器就会按最大的length分配空间。）

    

    3.4.2.2 sh_flags 字段

    SHF_ALLOC: 表示此节区在进程执行过程中需要占用内存。某些(用于)控制(的)节区并不加载进目标文件的内存映像中，则其sh_flags位应设置为 0。

    3.4.2.3

    表 10 sh_link和sh_info字段解释

    SHT_DYNAMIC sh_link: 此节区中条目所用到的字符串表(所在)的节区号 sh_info:0

    SHT_HASH sh_link: 此哈希表所适用的符号表(所在)的节区号 sh_info:0

    SHT_REL/SHT_RELA sh_link: 相关联的符号表(所在)的节区号 sh_info: 需要重定位操作的节区的节区号。表示该节区里包含有地址不明确的内容(变量/或函数符号地址不明确)，需要进行重定位操作。

    SHT_SYMTAB/SHT_DYNSYM sh_link: 相关联的字符串表(所在)的节区号 sh_info: 所有局部符号(LOCAL)的个数 + 1

    

    (用readelf –a ./test 查看：.rel.plt节区 的sh_link=4 对应符号表 .dynsym section；sh_info=b 是16进制，对应10进制的11，对应 .plt section)

    

    3.6 符号表（Symbol Table）

    表 13 符号表项字段

    st_shndx 一般是每个符号表项所对应的节区号。但某些数值具有特殊含义。参见3.6.3

    3.6.2 符号类型

    在共享目标文件(*.so)中，类型为

    STT_FUNC的函数符号具有特别的重要性。当其他目标文件引用了另1个共享目标文件(*.so)中的函数时，链接器(/bin/ld)自动为所引用的

    符号创建过程链接表项(plt项)。在共享目标文件(*.so)中，类型不是 STT_FUNC的函数符号不会自动通过过程链接表进行引用。

    

    3.6.4 符号取值

    不同的目标文件类型中符号表项对 st_value 成员具有不同的解释：

    在可重定位文件中，st_value 中遵从了节区索引为 SHN_COMMON 的符号的对齐约束。

    

    在可重定位文件中，st_value 是已定义符号的在本节区内的偏移量。就是说，st_value 是 st_shndx 所标识的节区内，从节区头部开始到符号位置的偏移。

    在可执行和共享目标文件中，st_value 是一个虚拟地址。为了让动态链接器更好的使用文件里这些符号的值，st_value不再使用节区内的偏移量表示（是针对文件的描述视图），而使用虚拟地址（是针对内存的描述视图），因为这时与节区无关。

    

    3.7.1 重定位表项

    表 17 重定位表项字段说明

    r_offset

    表示重定位操作后，需要填写修改的具体位置(地址或者偏移量)。对于一个可重定

    位文件(*.o)而言，此值是所在节区内的偏移量-表示本节区在此偏移量处的存储单元里包含的内容是要被重定位修改的。对于可执行文件(*.exe)或者

    共享目标文件(*.so)而言，此值是个内存虚拟地址，表示此地址处的存储单元里包含的内容是要被重定位修改的(一般这些虚拟地址都指向内存中的.got

    表里)。

    

    r_info

    表示要进行重定位操作的(变量/函数)符号

    在符号表(一般是.dynsym符号表)中的索引，以及将实施的重定位类型。对 r_info 成员使用 ELF32_R_TYPE

    宏运算可得到重定位类型，使用 ELF32_R_SYM 宏运算可得到符号在符号表里的索引值。

    #define ELF32_R_SYM(i) ((i)>>8)

    #define ELF32_R_TYPE(i) ((unsigned char)(i))

    #define ELF32_R_INFO(s, t) (((s)

    

    note：重定位的4个属性决定了重定位的具体位置和重定位类型。

    a. 重定位表.rel.text section的2个属性：

    sh_link: 表示需要重定位的符号属于哪个符号表。 sh_info: 表示要修改的节区。也即需要对哪个section里的内容进行重定位处理。

    b. 重定位表项的2个属性：

    r_offset: 表示重定位计算出的最终地址，要填写到哪里。 r_info: 表示要重定位的符号在符号表里的索引，以及将实施的重定位类型。

    note：重定位大致分2步，1. 对哪些内容 进行重新地址定位。 2.

    重定位操作完成后，把结果填写到哪里保存。不同的重定位类型

    情况会有所不同，一般重定位计算出的地址会填写到.got表里，以供以后符号解析使用，以后再使用这个符号时，直接查.got表就知道其最终地址，不需要

    浪费时间重定位该符号的地址了。

    3.7.2 重定位类型

    重定位表项描述如何对指令和数据字段里的地址/指针内容进行修改。一般，共享目标文件（*.so）在创建时，其基本虚拟地址是0，但在内存中的执行地址将随着动态加载而发生变化。

    3.8.1.2 基地址(Base Address)

    基地址用来对程序的内存映像进行重定位。

    可执行文件或者共享目标文件的基地址在执行过程中从3个数值计算的：内存加载地址、最大页面大小、程序的可加载段的最低虚地址。

    

    程序文件头部中的虚拟入口地址（entry point）不1定是程序在内存中映像的实际虚地址。要计算内存中映像的基地址，首先要找到所有

    PT_LOAD 段的最低 p_vaddr

    值所对应的的内存地址。按最大页面大小的整数倍，对内存地址进行取整截断，取最接近的数值就可以得到基地址。根据文件的不同类型，加载进内存的地址可能与

    p_vaddr 相同也可能不同。

    

    如前所述，“.bss”节区的类型为 SHT_NOBITS。尽管它在文件中不占据空间，但在内存映像里却会占用空间。通常，这些未初始化的数据位于段(segment)在内存中的映像的末尾，所以 p_memsz 会比 p_filesz 大。

    3.8.2 程序加载

    主要技术

    * 程序头部（Program Header）：描述与可执行程序执行直接相关的目标文件结构信息。用来定位各个段在文件中的位置。同时包含一些为可执行程序创建内存进程映像所必需的信息。

    * 程序加载：给定一个目标文件，操作系统加载该文件到内存中，启动程序执行。

    * 动态链接：操作系统加载了文件以后，必须对构成进程的各个目标文件之间的符号引用进行解析，以便完整地构造进程映像。

    当进程在执行过程中真正引用到相应的逻辑页面，操作系统才会请求真正的物理页面。通常进程会包含很多在执行中从来未引用过的逻辑页面，

    因此，操作系统用这种延迟物理读操作会避免很多不必要的物理页面读取工作，从而提高系统性能。但要想实际获得这种效率，可执行文件和共享目标文件的段

    (segment)必须符合如下条件：段在文件内的偏移量和段在内存中的虚拟地址 对页面大小取模后余数相同。

    在这个例子中，文件的4个页面里包含的数据不是纯粹属于正文段或者数据段。（每个页面大小是0x1000=4096）

    (1). 文件的第一个页面中包含了 ELF 头部、程序头部表、以及其它信息

    (2). 文件的最后一个页面包含数据开始部分的一个副本

    (3). 数据段的第一个数据页面里包含了正文段的末尾部分

    (4). 数据段的最后一个数据页面里可能包含与运行进程无关的文件信息

    

    不过系统对这些页面一般会做两次映射，以保证每个段的内存访问读写许可权限是相同的。加载到内存里时，数据段的末尾需要对未初始化数据(.bss section)进行特殊处理，操作系统应该将这些数据初始化为0。

    

    可执行文件与共享目标文件的段加载进内存有一点不同。可执行文件的段里通常都是绝对地址的代码，为了能够让进程正确执行，段加载进内存所使用的段地址必须是连接器创建可执行文件时所使用的虚拟地址。因此系统会使用p_vaddr作为虚拟地址。

    

    共享目标文件（*.so）的段里通常都是位置无关的代码。这使得在不同的进程内存映像中，共享目标文件的段的内存虚拟地址也不同，但并不影响其

    正常执行。尽管操作系统为每个进程内存映像分配独立的虚拟地址空间，仍能维持段的相对位置。位置独立的代码在段(代码段)与段(数据段)之间使用相对寻

    址，内存里虚地址之间的偏移量必须与文件中虚拟地址之间的偏移量相匹配。

    

    下表给出同1个共享目标文件(*.so)在不同进程内存映像中的虚拟地址方案，说明了这种重定位问题。

    

    3.8.3 动态链接

    3.8.3.1 程序解释器

    可执行文件中可以包含PT_INTERP类型的程序头表项目。在系统调用

    exec()执行过程中，操作系统从PT_INTERP段中检索解释器文件的路径名(例如：/lib/ld-linux.so)，用解释器文件创建初始的

    进程映像。也就是说，操作系统并不使用原来可执行文件的段映像，而是用解释器构造一个内存映像。接下来是解释器从操作系统接收控制，为真正的应用程序创造

    执行环境。

    

    解释器可以有两种方式接收控制：

    * 操作系统为解释器提供一个文件描述符，解释器读取可执行文件并将其映射到内存中。

    * 根据可执行文件的格式，操作系统可能把可执行文件加载到内存中，而不是为解释器提供一个已经打开的文件描述符。

    

    解释器可以是一个可执行文件，也可以是一个共享目标文件(例如：/lib/ld-linux.so)。

    

    可共享目标文件(正常的情形)是位置无关地加载进内存，在不同进程中的(加载)地址也各不相同；系统把可共享目标文件的段的内存映像创建在动态

    的段区域中，动态段区域被mmap(KE_OS) 和相关服务例程

    所使用。因而，如果解释器是一个共享目标文件，解释器本身的加载地址将不会和(要加载的)可执行文件的初始段地址发生冲突。

    

    可执行文件被加载到内存中固定地址，系统使用其程序头表项中的虚拟地址在内存中创建各个段映像。因此，可执行文件类型的解释器的虚拟地址可能会与(要加载的)可执行文件的虚拟地址发生冲突。解释器要自己负责解决这种冲突。

    

    3.8.3.2 动态加载程序

    连接器在构造创建使用动态链接库的可执行文件时，向可执行文件中添加一个类型为PT_INTERP的程序头表项，告诉系统要使用动态链接器作为程序解释器。动态链接器的具体路径位置是和操作系统相关的。

    

    3.8.3.4 共享库的依赖关系（共享库=动态链接库 *.so）

    当连接器(/bin/ld)处理一个静态库(*.a)时，它取出库

    里相关内容并且把它们拷贝到输出文件(*.so或*.exe)中。这些被静态连接的服务例程函数在程序执行期间是直接可用的，不需要动态链接器的参与。共

    享库文件也提供了服务例程函数，动态连接器必须把适当的共享库内容放在进程映象中以便执行。因此，可执行文件和共享库文件记述了相互之间明确的依赖关系。

    

    当动态连接器(/lib/ld-linux.so)为一个object文件创建 内存段

    时，依赖关系(记录在动态连接数组中的d_tag=DT_NEEDED类型的多个数组项中)说明需要哪些动态链接库文件来为程序提供服务例程函数。依照这

    些被引用的动态链接库文件和他们的依赖关系(#

    用ldd程序可以读取并显示这些依赖关系)，动态连接器（1次连接1个动态库）通过多次连接建立起一个完整的进程映象。当解析符号引用的时候，动态连接器

    以宽度优先搜索(算法)来检查符号表，也即，动态连接器先查看可执行程序自身的符号表，然后查看

    DT_NEEDED类型动态连接数组项中记录的“动态库文件”的符号表(按顺序)，再接下来查看第二级DT_NEEDED类型动态连接数组项记录的“动态

    库文件”的符号表，依次类推。动态库文件必须对进程是可读的；其他权限则不是必需的。

    即使某个共享库文件在依赖表中出现多次，动态链接器也仅会对其连接一次。



下图是readelf工具输出的某ELF文件segments与sections信息：

 segments部分包含各segments的地址、偏移、属性等；而sections部分则依次列出每个segment所包含的sections。注意到，sections通常为全小写字母，而segments通常为全大写字母。

此外，这两者之间另一个关键不同点是，sections包含的是链接时需要的信息，而segments包含运行时需要的信息。即，在链接时，链接器通过section header table去寻找sections；在运行时，加载器通过program header table去寻找segments。可见下图：

 

一些比较重要的sections如下：

(1) .init_array: 动态库加载或可执行文件开始执行前调用的函数列表
(2) .text: 代码
(3) .got: Global offset table(GOT)，包含加载时需要重定位的变量的地址
(4) .got.plt: 包含动态库中函数地址的GOT
一些比较重要的segments如下：

(1) LOAD: 运行时需要被加载进内存的segment
(2) GNU_STACK: 决定运行时栈是否可执行
(3) DYNAMIC: 动态链接信息，对应于.dynamicsection
1.2常用工具

在静态、动态分析ELF文件时，经常用到以下工具：

(1) ptrace    

#include <sys/ptrace.h>
 long ptrace(enum __ptrace_requestrequest, pid_t pid,
                 void *addr,void *data);
系统调用。用于监控其他进程，被gdb, strace, ltrace等使用

(2) strace

命令行工具。用于追踪进程与内核的交互，如系统调用、信号传递

(3) ltrace

命令行工具。类似于strace，但主要用于追踪库函数调用

(4) readelf/objdump

静态分析工具。用于读取ELF文件信息。


    
这个ebp也不是必须的，实际esp虽然不停在变，但具体变化编译器是可以编译其计算出来的，因此直接使用esp来访问局部变量和参数也是可行的。gcc有个编译参数 -fomit-frame-pointer 就是干这个事的

nm
·   The symbol type.  At least the following types are used; others are, as
           well, depending on the object file format.  If lowercase, the symbol is
           usually local; if uppercase, the symbol is global (external).  There are
           however a few lowercase symbols that are shown for special global
           symbols ("u", "v" and "w").

           "A" The symbol's value is absolute, and will not be changed by further
               linking.

           "B"
           "b" The symbol is in the uninitialized data section (known as BSS).

           "C" The symbol is common.  Common symbols are uninitialized data.  When
               linking, multiple common symbols may appear with the same name.  If
               the symbol is defined anywhere, the common symbols are treated as
               undefined references.

           "D"
           "d" The symbol is in the initialized data section.

           "G"
           "g" The symbol is in an initialized data section for small objects.
               Some object file formats permit more efficient access to small data
               objects, such as a global int variable as opposed to a large global
               array.

           "i" For PE format files this indicates that the symbol is in a section
               specific to the implementation of DLLs.  For ELF format files this
               indicates that the symbol is an indirect function.  This is a GNU
               extension to the standard set of ELF symbol types.  It indicates a
               symbol which if referenced by a relocation does not evaluate to its
               address, but instead must be invoked at runtime.  The runtime
               execution will then return the value to be used in the relocation.

           "I" The symbol is an indirect reference to another symbol.

           "N" The symbol is a debugging symbol.

           "p" The symbols is in a stack unwind section.

           "R"
           "r" The symbol is in a read only data section.

           "S"
           "s" The symbol is in an uninitialized data section for small objects.

           "T"
           "t" The symbol is in the text (code) section.

           "U" The symbol is undefined.

           "u" The symbol is a unique global symbol.  This is a GNU extension to
               the standard set of ELF symbol bindings.  For such a symbol the
               dynamic linker will make sure that in the entire process there is
               just one symbol with this name and type in use.

           "V"
           "v" The symbol is a weak object.  When a weak defined symbol is linked
               with a normal defined symbol, the normal defined symbol is used with
               no error.  When a weak undefined symbol is linked and the symbol is
               not defined, the value of the weak symbol becomes zero with no
               error.  On some systems, uppercase indicates that a default value
               has been specified.

           "W"
           "w" The symbol is a weak symbol that has not been specifically tagged as
               a weak object symbol.  When a weak defined symbol is linked with a
               normal defined symbol, the normal defined symbol is used with no
               error.  When a weak undefined symbol is linked and the symbol is not
               defined, the value of the symbol is determined in a system-specific
               manner without error.  On some systems, uppercase indicates that a
               default value has been specified.

           "-" The symbol is a stabs symbol in an a.out object file.  In this case,
               the next values printed are the stabs other field, the stabs desc
               field, and the stab type.  Stabs symbols are used to hold debugging
               information.

           "?" The symbol type is unknown, or object file format specific.

       ·   The symbol name.
