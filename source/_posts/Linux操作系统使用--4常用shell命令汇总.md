---
title: Linux操作系统使用--4常用shell命令汇总
date: 2017-10-18 09:21:20
categories: 嵌入式
tags: [嵌入式,学习笔记,shell命令,环境变量]
comments: true

---

>本篇文章从基本命令、磁盘管理、用户管理、进程管理、文件管理等几个方面总结实践中shell常用命令及其使用场合。并在最后简要介绍环境变量的相关知识：定义、查看、修改。<!-- more -->

# shell简介
## 基本概念    

- shell是一个命令行解释器，其功能是将用户命令解析为操作系统所能理解的指令，实现内核和用户的交互。
- shell命令包括三个要素，命令名称、选项、参数。命令名称区分大小写。
- 当多个shell命令写在一行，用分号隔开，顺序执行每一个命令。
- 当一条shell命令一行写不完，用反斜杠表明。

## shell中的特殊字符
### 通配符
当用shell命令处理一组文件时，通常使用通配符，提高效率。注意和正则表达式区分。常用于文件名匹配。

	- * 匹配任意长度字符
	- ? 匹配一个长度字符
	- […] 匹配其中指定的一个字符
	- […-…] 匹配指定的一个字符范围
	- [^…] 除了指定的字符，其他均可匹配 

### 管道
功能：将第一个命令的输出作为第二个命令的输入（单竖线 |）
示例：
ps -ef | grep xxx 用于杀死某个进程时，先用该命令定位进程号
ls | wc -l 统计文件个数
### 置换
功能：将一个命令的输出作为另一个命令的参数（Esc下方的引号）
示例： 
wc -l \`ls\` 统计每个文件的行数
ls \`pwd\` 显示当前路径文件 
### 重定向
功能：分为输入、输出、错误重定向，输出重定向用的多。主要是改变shell命令或者程序默认的标准输入、输出、错误目标。
示例：
\> 新建模式，覆盖原文件
\>\> 追加模式，写在已有内容后
*注：使用\> filename可以新建文件* 
*参考文档：[shell中的通配符，特殊字符和正则表达式][1]*
# 常用shell命令
## 基本命令
这里只简要总结常用shell命令。

- whereis xxx 查找二进制文件、可执行文件、帮助文档，源文件、配置文件等。
- which xxx 查找可执行文件所在位置。
- hostname 查看主机名。
- whoami 查看当前用户。
- history numberline 显示历史命令，在重启电脑后可以查看，以便执行过去的命令。
- wc \-c/\-w/\-l xxx 统计文件字符数/单词数/行数。

*参考文档：[四个查找命令find,locate,whereis,which的区别][2]*
*注：掌握某些命令的选项含义，能够举一反三套用在多个命令中，例如\-r表示 recursion递归，\-h表示human人类可读。*

## 磁盘管理
- df -aTh 查看文件系统空间占用磁盘情况，显示文件格式类型，在清理磁盘时常用。
- du -h 查看某个目录/文件所占磁盘空间大小。
- dd if=xxx of=xxx bs=xxx count=xxx 块拷贝文件，烧写系统时将镜像拷贝到优盘用到；制作ramdisk文件系统用到。
- fdisk 查看硬盘分区及对硬盘进行分区管理。ITOP4412烧写系统用到fdisk -c 0。
- mount \-t types device mountpoint 将设备文件挂载在某个目录下，openwrt下有手动挂载和自动挂载使用；制作ramdisk文件系统用到。
- umount mountpoint 将设备文件卸载。

## 用户管理  
- adduser username 添加用户。
- passwd username 修改用户的密码。
 
*注：*
*1. 添加一个用户时，系统会将/etc/skel目录下的文件、目录都复制到新用户的主目录下，主要是一些配置文件，如.vimrc、.bashrc等。且会保存在/etc/passwd文件里，一行包含一个账号信息。此外，/etc/group文件记录了组的名称和组员列表。*
*2. 如果忘记密码，可以使用启动界面进入Recovery Mode，以root身份修改其他用户密码。也可以使用passwd直接修改其他用户密码，不必输入旧密码。*

## 进程管理
- ps aux 查看进程，侧重查看进程的CPU占用率和内存占用率。
- ps -ef 查看进程，侧重查看进程的父进程ID和完整的COMMAND命令。
- pstree 以树状形式显示进程。

*注：*
*1. 其余关于进程的状态等知识点另起专题研究。*
*2. ps命令经常搭配grep命令，关注感兴趣的进程。*

## 文件管理
### 文件及文件夹
- pwd 查看当前路径，在需要粘贴绝对路径时使用该命令。
- file xxx 查看文件类型。 
- rm -rf 强制删除，包括文件夹。小心使用该命令，尤其是root用户。一般不使用rmdir，该命令要求必须为空文件夹。
- mkdir -p dir1/dir2 创建多级目录。
- head -num xxx 显示文件前num行内容，通常搭配其他命令结合管道使用。
- tail -num xxx 显示文件后num行内容。
- grep pattern file 查找字符串，格式支持**正则表达式**。
- find path -name "xxx" 查找文件,支持通配符**\*和?**。
- diff a b 比较a和b两个文件。

### 创建链接
- ln target link\_name 创建硬链接，通过每个文件的iNode建立，不能跨越文件系统。会在选定位置生成一个和源文件大小相同的文件。
- ln -s target link\_name 创建软链接（符号链接），利用文件路径名建立，使用绝对路径，可以跨越文件系统。实践中使用较多。不会重复占用磁盘空间。

### 压缩与解压
- tar -cvjf xxx.tar.bz directoryname 压缩为bz格式。
- tar -cvzf xxx.tar.gz directoryname 压缩为gz格式。
- zip xxx.zip directoryname 压缩为zip格式。
- tar -xvjf xxx.tar.bz 解压bz格式。
- tar -xvzf xxx.tar.gz 解压gz格式。
- unzip xxx.zip 解压zip格式。

*注：*
*1. iNode是检索i节点表的下标，i节点中存放有文件的状态信息。*
*2. 修改硬链接文件名，链接仍然有效，而软链接断开连接。对于已存在的链接执行移动或删除命令，可能会导致链接断开。如果删除后，建立同名文件，软链接将恢复，但硬链接不再有效。*

## 网络管理
网络管理主要是配置上网。临时生效使用命令配置，永久生效编辑配置文件。关于网络的其他基础知识和常用的命令另起专题研究。

- ifconfig 查看网络。
- ifconfig eth1 192.168.1.126 netmask 255.255.255.0 配置IP和子网掩码。

**Ubuntu/Raspbian** 
网卡配置文件：/etc/network/interfaces
dns配置文件：/etc/resolv.conf
重启生效 ：sudo /etc/init.d/networking restart

**openwrt**
网卡配置文件：/etc/config/network
重启生效：/etc/init.d/network reload

*注：*
*1. 如果ping不通域名网址，可能需要修改dns配置文件。*
*2. resolv.conf文件在重启后被清空，需要再次配置。*

# 环境变量
环境变量是指用户运行环境的参数集合。每一个用户都有其专门的运行环境。用户可以对运行环境进行定制，即修改系统环境变量。环境变量的书写为 环境变量名=内容1：内容2。使用环境变量的场合有：

1. 安装软件，配置路径环境变量，以便全局使用。
2. 切换用户时，通常带上环境变量和工作目录，确保软件的正常使用，如su - root。

## 常见的环境变量
- PATH 系统路径
- HOME 系统家目录
- HISTSIZE 保存历史命令记录的条数
- LOGNAME 当前用户的登录名
- HOSTNAME 主机名称
- SHELL 当前用户用的哪种shell

## 查看/设置环境变量
- env 查看所有环境变量。
- echo $environmentVariable 查看某一环境变量的内容。
- export PATH=/home/youbo:$PATH 临时添加环境变量，只对当前终端有效。

如果想要永久性修改环境变量，则需要修改配置文件。

- /etc/profile 系统级环境变量

e.g:安装java和tomcat

- /etc/bash.bashrc 系统级bashrc

e.g:安装openwrt交叉编译环境

- /home/xxx/.bashrc 用户级环境变量

e.g:修改HISTSIZE

*注：修改完配置文件后，需要使用source命令使其生效，不必重启*
*参考文档：[Linux环境变量种类、文件、设置][3]*

[1]: http://blog.csdn.net/miss_acha/article/details/24462519
[2]: http://www.cnblogs.com/IvanChen/p/5179507.html
[3]: http://www.linuxidc.com/Linux/2013-01/78000.htm