---
title: C语言高级编程--2搭建嵌入式linux编程环境
date: 2017-11-17 09:21:15
categories: 嵌入式
tags: [嵌入式,学习笔记,gcc,gdb,交叉编译]
comments: true

---
>搭建嵌入式Linux编程环境，就是搭建C语言编程环境，这是嵌入式系统开发的首要工作。另外在嵌入式Linux系统移植中，也需要首先搭建交叉开发环境。嵌入式开发工具包括编辑器、编译器、调试器、工程管理器等。前面已介绍过VIM编辑器，本篇主要总结gcc编译器和gdb调试器，此外简要介绍GNU Binutils工具集，并给出实践的使用实例。工程管理器make将在下一篇总结。<!-- more -->

# GNU工具集
GNU工具集是包含了GNU项目所产生的各种编程工具的集合。这些工具形成一条工具链，用于开发应用程序和操作系统。ＧNU工具有很多，包括如下：

- 编译工具：gcc。
- 调试工具：gdb。
- 二进制工具集：GNU Binutils,包括链接器、汇编器、格式转换工具等。
- 软件管理工具：make、CVS、subvision等。

*参考文档：*
*[gnu工具集][1]*
*[GNU工具链（GNU toolchain）][2]*
*[gnu工具链简介 ][3]*

## GNU Binutils
binutils工具集主要包括as汇编器和ld链接器，除此之外，还有一些工具(即命令)是开发调试中常用的，此处做一简要介绍。
```
readelf -h xxx 查看可执行文件的头部信息，包括系统架构、大小端等信息。elf文件是linux/unix系统下的二进制文件格式。

readelf -a xxx 查看可执行文件的所有信息。

file xxx 查看文件类型，如果是可执行文件，会显示文件类型、系统架构、大小端、是否strip等信息。

size xxx 列出可执行文件每一段的大小和总大小。可显示可执行文件包括text、data、bss等部分。

strip xxx 对可执行文件“瘦身”，丢弃目标文件中的全部或特定符号，减小文件体积。对于嵌入式系统，需要使用该命令。不可对中间文件“瘦身”，否则链接出错。

nm xxx 查看可执行文件的所有符号。在调试段错误时可以使用，方便定义哪里发生了段错误。

objdump -d xxx 对可执行文件进行反汇编，生成汇编代码，在调试段错误也会用到。
 
objcopy 进行目标文件格式转换 例如：objcopy --gap-fill=0xff -O binary u-boot u-boot.bin,用在uboot移植部分。

addr2line 把程序地址转换为文件名和行号，常用于内核调试。注意编译时需加上-g选项，生成调试符号。否则运行addr2line出错。
e.g
gcc -o test -Wl,-Map=test.map -g test.c
grep main test.map
addr2line 0x08048368 -e test -f
```

*参考文档：*
*[Binutils工具集 解析][4]*
*[Linux环境下段错误的产生原因及调试方法小结][5]*
*[Linux debug : addr2line追踪出错地址][6]*

## GCC
### GCC简介
GCC可以编译如C、C++、Java等多种语言，GCC是可以在多种硬件平台上编译出可执行程序的超级编译器，其执行效率比一般的编译器要高20%-30%。在嵌入式领域开发需要交叉平台编译器。

### GCC的基本用法
该部分介绍GCC命令常用的选项，需熟练掌握。因为在makefile中也就是使用这些语句组成。

```
-c 只编译,不链接为可执行文件。用于编译不包含主程序的子程序文件。由.c文件生成.o文件。

-g 在使用gdb之前需要加上该选项。

-o output_name 指定输出文件名。

-O/O2 优化编译链接过程。-On n表示优化级别的整数。优化可以加快代码运行速度，但是会对调试造成影响，因此在开发阶段不要进行优化，在发布时可以优化。

-I dirname 将dirname所指向目录加入到程序头文件目录列表中,在预编译过程使用。

-L dirname 将dirname所指向的目录加到程序函数档案库文件的目录列表中,在链接时用到。

-llibrary 链接名为library的库文件。

-Dname 预定义一个名为name的宏，值为1。也可以是-Dname=xxx，值为xxx。 可以用于调试，如-DDEBUG。

-Wall 打开所有类型的语法警告。运行含有警告信息的代码通常会有意想不到的结果，所以应尽量使用-Wall处理警告信息。

-std=c99 指明使用标准ISO C99作为标准编译程序。

-static 禁止使用动态库。

-share 尽量使用动态库。

```
### GCC编译过程
GCC编译包括4个步骤：

1. 预处理
命令：gcc -E test.c -o test.i
作用：进行编译的第一遍扫描。由预处理程序负责完成。当编译一个程序时，系统自动调用预处理程序对程序中的以＃开头的预处理部分进行展开，然后才进入源程序的编译阶段。包括替换宏、条件编译、头文件展开等。该阶段不检查语法错误。
2. 编译
命令：gcc -S test.i -o test.s
作用：检查代码的规范性，是否有语法错误，检查无误后把代码翻译为汇编代码。
3. 汇编
命令：gcc -c test.c -o test.o 或者gcc -c test.s -o test.o或者 as test.s -o test.o
作用：生成目标代码。
4. 链接
命令：gcc test.o  -o test
作用：将目标程序链接库资源，生成可执行程序。对于头文件包含的函数声明，其函数实现大多数都包含在libc.so.6这个函数库中，gcc默认在路径/usr/lib下去搜索，即链接到libc.so.6库函数中，从而找到函数的实现。这就是链接的作用。

## GDB
### GDB使用
GDB是GNU开源组织发布的一个强大的UNIX下的程序调试工具。使用时，必须在编译时加上-g选项，才可以使用GDB调试。编译完成后，使用命令gdb xxx,即进入调试过程。以下列出常用的选项。
```
l 查看文件。

b n 在第n行设置断点，表示程序运行到第n行之前停止。

r 运行代码。

c 恢复代码运行。

p xxx 打印变量值xxx，在显示变量值时前面有一个$N,表示当前变量值的引用标记，后续再次查看该变量，可以使用$N简化书写。

n 非进入式单步运行。

s 进入式单步运行。

watch xxx 对xxx变量设置观察点，单步运行会看到变量值的变化。

q 退出。
```

*参考文档：*
*[Linux gdb调试器用法全面解析][7]*

# 嵌入式交叉开发环境搭建
搭建嵌入式交叉开发环境是进行嵌入式应用开发前必需的准备工作，也是嵌入式系统移植的第一步。开发环境包括主机、目标机、以及连接介质。搭建嵌入式交叉开发环境就是要在主机上编译、调试目标机的代码，即涉及主机和目标机的通信。
关于嵌入式交叉开发环境搭建有以下几点说明：

- 理解为什么要交叉编译。交叉编译是指在开发主机上运行编译器编译内核、应用程序，在目标机上运行内核和应用程序。原因是嵌入式系统硬件资源限制，且嵌入式系统MCU体系结构和指令集不同。
-  理解如何搭建嵌入式开发环境。
	1. 准备好主机、目标机和连接介质。
	2. 准备好目标机代码。
	3. 安装交叉工具链。
	4. 准备主机和目标机之间通信的辅助工具，即安装通信服务和软件。
- 关于安装交叉工具链。主要步骤是获取交叉工具链压缩包、解压、添加全局环境变量到配置文件、source 配置文件。一般情况是从网上下载已经编译好的交叉工具链。也可以下载源码，自行编译交叉工具链，需要使用工具crosstool-ng，后续有需求再做深入了解。GNU 交叉工具链和前面叙述一致，包括GNU Binutils、GNU GCC、GNU Glibc等部分。
- 连接主机和目标机的介质通常是串口、USB、网络。对于每一种方式，掌握使用什么工具，如何使用，有何作用即可。具体情况需结合实践情况进行分析。通常情况下，有以下两个任务需要完成。
	- 进入目标机终端的方法：（1）使用串口。（2）使用网络协议ssh、Telnet等。不管是使用串口还是网络，都可以使用putty、scureCRT软件，因为其支持serial、ssh、ftp等众多协议。此外，也可以使用命令行远程控制，例如ssh root@192.168.1.105。
	- 传输目标机代码的方法：（1）使用串口。使用dnw或超级终端软件。（2）使用网络协议scp、tftp等传输。同样，可以使用secureCRT，winSCP(windows平台下使用)等软件，也可以使用命令行，例如在主机上运行命令scp test root@192.168.1.105:/上传文件，或者在开发板上运行scp youbo@192.168.1.109:/home/youbo/test .下载文件。
	- 在使用的嵌入式硬件平台中，（1）JS9331开发板，使用串口进入终端，使用scp传输文件。（2）树莓派和Intel NUC3815，使用ssh进入终端，使用scp传输文件。（3）FS4412/ITOP4412开发板，使用串口进入终端，使用tftp传输文件，使用nfs共享根文件系统，此外还使用dnw或者超级终端烧写程序。

**说明**：使用网络协议nfs共享根文件系统，是开发调试时比较方便的一种方法。应该理解为根文件系统在主机上，主机上对根文件系统的操作会同步到目标机上。因此可以方便的拖动应用软件，而不需要传输。等待功能开发完毕后，可以将根文件系统压缩传输到目标机的flash中，启动时从flash读取根文件系统即可。具体内容还可参见系统移植部分。
 
*参考文档：*
*[linux：嵌入式linux开发环境搭建（整理）][13]*
*[嵌入式Linux开发环境的搭建之：嵌入式开发环境的搭建 ][14]*
*[NFS文件系统简介及原理][15]*
*[目标板通过nfs挂载根文件系统][16]*
*[嵌入式CRT和DnW工具有什么区别][12]*
*扩展文档：*
*[OpenWrt NFS启动 ][17]*

# 实例分析
## GCC编译C++程序
在实际编程中可能涉及调用第三方库函数。而基于第三方库，项目中他人可能编写c++程序，因此涉及如何同时编译c程序和c++程序。以openwrt移植opencv库，并使用c程序调用c++程序函数为例进行分析。详情参见[源代码和Makefile][21]。

说明：

1. 使用gcc编译C程序，使用g++编译C++程序，使用g++或者gcc -lstdc++链接。 
2. 如果要使用C程序调用C++函数，对于C++函数形参中的对象变量如何在C程序转换使用，需要扩展学习。因此此段代码暂时使用IplImage *指针传参，在函数内部转换为Mat类型。

*参考文档：*
*[linux下C与C++混合编程 ][8]*
*[GCC编译C C++ 和C混合C++][9]*

## GDB远程调试 
在实际开发过程中，调试程序是必不可少的环节，对于应用程序的调试，可以采用print方法。也可以使用gdb方法。因为gdb通常很大，不适合直接在目标机中运行，因此通常使用目标机运行gdbserver,主机运行gdb进行远程调试。此外，还有一些其他方法，例如分析core文件调试段错误，后续根据需要可做扩展研究。
这里分析gdb远程调试的方法。

### js9331开发板
使用gdb远程调试包括以下步骤：

1. 在openwrt源码目录下make menuconfig选中toolchain option中的gdb,以及development中的gdbserver。
2. 编译openwrt。将/openwrt/staging_dir/toolchain-mips_34kc_gcc-4.8-linaro_uClibc-0.9.33.2拷贝到/opt目录下，配置好环境变量。
3. 将/openwrt/bin/ar71xx/packages/base/目录下的gdbserver包及其依赖包传输到开发板进行安装。
4. 开发板运行命令./gdbserver 192.168.1.111:1234 hello。
5. 主机运行mips-openwrt-linux-gdb hello，进入gdb环境。
6. 在gdb环境下运行target remote 192.168.1.105:1234。
7. 在gdb环境下查看搜索文件路径前缀show solib-absolute-prefix，并设置为交叉编译环境库路径前缀set solib-absolute-prefix /opt/OpenWrt-Toolchain-ar71xx-for-mips_34kc-gcc-4.8-linaro_uClibc-0.9.33.2/toolchain-mips_34kc_gcc-4.8-linaro_uClibc-0.9.33.2/。之后则进入正式调试环节。

### ITOP4412开发板
使用gdb远程调试包括以下步骤：

1. 在arm-2009q3交叉工具链中找到gdbserver可执行文件，使用arm-none-linux-gnueabi-strip对其进行瘦身。
2. 使用tftp将瘦身后的gdbserver传输到开发板。
3. 使用arm-none-linux-gnueabi-gcc编译待调试代码，加入选项-g。并用tftp传输到开发板。
4. 开发板运行命令./gdbserver 192.168.1.111:1234 hello。
5. 主机运行arm-none-linux-gnueabi-gdb hello，进入gdb环境。
6. 在gdb环境下运行target remote 192.168.1.230:1234。
7. 在gdb环境下查看搜索文件路径前缀show solib-absolute-prefix，并设置为交叉编译环境库路径前缀set solib-absolute-prefix /opt/arm-2009q3/arm-none-linux-gnueabi/libc。之后则进入正式调试环节。

*参考文档：*
*[嵌入式开发调试方法 ][10]*
*[嵌入式Linux的调试技术][11]*
*[使用gdbserver远程调试][18]*
*[openwrt下使用gdbserver远程调试][19]*

## 搭建交叉开发环境 
搭建交叉开发环境，主要是安装交叉编译工具链。此外，为了调试方法，会使用tftp和nfs服务，下面做简要分析。这里的操作均是在主机上运行。

### js9331开发板
openwrt系统搭建交叉编译环境步骤如下：

1. openwrt官网下载工具链压缩包。
2. 解压到/opt文件夹下。
3. 配置环境变量，修改/etc/bash.bashrc文件。
4. 执行命令source /etc/bash.bashrc。
5. 执行mips-openwrt-linux-gcc -v检查安装是否成功。

### ITOP4412开发板
linux最小系统搭建交叉编译环境步骤如下：

1. 获得工具链压缩包。
2. 解压到/opt文件夹下。
3. 配置环境变量，修改/etc/bash.bashrc文件。
4. 执行命令source /etc/bash.bashrc。
5. 执行arm-none-linux-gnueabi-gcc -v检查安装是否成功。

### 使用tftp服务

1. 主机安装tftpd-hpa和tftp-hpa。
2. 配置tftp server配置文件，/etc/default/tftpd-hpa。内容如下：
```
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="-c -s -l"
```
3. 创建/tftpboot文件夹，并改变权限。
4. 重启tftp服务，sudo /etc/init.d/tftpd-hpa restart。
5. 复制目标机代码到/tftpboot文件夹下。
6. 目标机输入命令"tftp -g -l 目标文件名 -r 源文件名 tftp服务器IP"即可下载代码。

### 使用nfs服务

1. 主机安装nfs-kernel-server。
2. 配置文件，/etc/exports。内容如下：
```
/source/rootfs *(rw,sync,no_root_squash,no_subtree_check)
```
3. 创建/source/rootfs文件夹，解压根文件系统到该目录。
4. 复制目标机代码到/source/rootfs文件夹下。
5. 配置目标机启动参数。

注：在集成开发环境也可以编译、调试程序，例如eclipse CDT,其中也需要设置编译器、调试器等参数，其原理本质上和gcc、gdb等一样。

*参考文档*
*[《嵌入式linux c语言程序设计基础教程》1.6][20]*

# 个人观点
这一部分不涉及知识难点，只需要会使用gcc、gdb，以及主机和目标机之间通信的工具，达到完成开发的目的即可。

[1]:http://blog.chinaunix.net/uid-20331961-id-3079782.html
[2]:https://www.cnblogs.com/MuyouSome/archive/2013/05/05/3061636.html
[3]:http://blog.csdn.net/u011630575/article/details/48676479
[4]:http://blog.csdn.net/zqixiao_09/article/details/50783007
[5]:http://blog.csdn.net/yuzeze/article/details/53144072
[6]: http://www.linuxidc.com/Linux/2011-05/35780.htm
[7]: http://blog.csdn.net/21cnbao/article/details/7385161
[8]: http://blog.csdn.net/weiyuefei/article/details/51065802
[9]: http://blog.csdn.net/junfeng_liu6/article/details/40075721
[10]: https://my.oschina.net/shelllife/blog/180901
[11]: http://www.cnblogs.com/xiansheng/p/5598994.html
[12]: https://zhidao.baidu.com/question/1755854973320265828.html
[13]: http://blog.csdn.net/sinat_36184075/article/details/71194832
[14]: http://www.eefocus.com/embedded/322804/r0
[15]: http://blog.csdn.net/waterfall_zjw/article/details/50957095
[16]: https://www.cnblogs.com/youthshouting/p/4541727.html
[17]: http://blog.csdn.net/walker0411/article/details/51944096
[18]: https://www.cnblogs.com/pengdonglin137/p/4737045.html
[19]: http://blog.csdn.net/zimiao815/article/details/51312813
[20]: http://www.ptpress.com.cn/shopping/buy?bookId=cd38272a-8d0e-4ff8-a02f-698d013ec3e2
[21]: https://gist.github.com/youxiaobo/b168b82d0192b90c7d54ab1b92956525