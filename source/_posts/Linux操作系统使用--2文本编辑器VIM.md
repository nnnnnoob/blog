---
title: Linux操作系统使用--2文本编辑器VIM
date: 2017-09-27 09:21:15
categories: 嵌入式
tags: [嵌入式,学习笔记,VIM]
comments: true

---
>本篇文章简要叙述文本编辑器VIM的使用，包括VIM的3种模式，常用的功能键，常用的插件配置。同时给出可供参考的IDE，以作扩展学习。<!-- more -->

# vim编辑器  

## 3种模式
- 命令模式：主要是光标跳转、复制、粘贴、删除。

- 编辑模式：主要是文字编辑。

- 底行模式：主要是保存、退出、查找、替换、设置编辑环境。

*注：三种模式叫法可能存在差异，理解每个模式下的功能即可*

## 常用功能键与快捷键
这里只列举常用的一些功能键和快捷键。

**多窗口与多标签**

 - 新窗口 Ctrl+shift+n
 - 新标签 Ctrl+shift+t
 
**跳转** 

- 跳转到第n行 ngg 
 
**查找字符串** 
 
- 正向查找 /str 
- 跳到下一处 n 
- 跳到上一处 N 
- 反向查找 ?str
 
**替换字符串**

- m行到n行的str1替换为str2 ：m,ns/str1/str2/g
- 全文str1替换为str2 ：%s/str1/str2/g 
 
 *注：最后的g表示一行中所有匹配的字符串，不加则只替换第一处*

**撤销与反撤销**

- 撤销 u 
- 反撤销 Ctrl+r 
 
**重复上一修改正文的操作**

- .(点号) 
 
**显示行号**

- :set number
 
**分屏**

 - 竖分屏 :vsp filename
 - 横分屏 :sp filename 
 
**比较两个文件**

 - 未打开文件时 vimdiff a.c b.c
 - 打开a.c时，和b.c比较 :vert diffsplit b.c
 
*参考文档：[Vim入门基础][1]* 
## 配置文件与插件
网上有许多关于VIM配置的说明，我认为最重要的是知道自己要做什么，而不是盲目的使用教程做复杂的配置。 
 
作为新手，我主要用vim进行C语言开发，我常用以下功能。
  
	- 查看定义(宏、变量、函数名等) 
	
	- 符号一览(宏、变量、函数名等) 
	 
	- 查看文件目录 
	
	- 代码补全
因此需要下载相关插件，并编辑配置文件，以期实现以上功能。其中插件下载后存放于~/.vim文件夹，配置文件为~/.vimrc，当前用户有效。

### ctags插件实现查看定义
使用步骤 

	1. 在源码目录下执行ctags -R，生成tags文件 
	
	2. 在.vimrc文件中添加:set tags +=源码路径/tags 
	 
	3. 跳转定义使用Ctrl+] 
	
	4. 返回使用Ctrl+t 
	
**注：程序修改后，需重新运行ctags -R**

### taglist插件实现符号一览
使用步骤 

	1. 前提：ctags已经打好标签 
 
	2. 打开taglist窗口使用:Tlist 
	 
	3. 光标选择符号，按enter键跳转到定义
	  
	4. 光标选择符号，按空格键，在底行显示tag的完整表达
	  
### netrw、WinManager插件实现查看文件目录
使用步骤
  
	1. 注：netrw已随vim安装，无需下载 
	 
	2. WinManager的作用是组合netrw窗口和taglist窗口
	  
	3. 打开窗口使用wm,关闭窗口再次使用wm
	
	4. 光标选择文件或文件夹，按enter键进入 
	
	5. 返回上级目录使用-键 
	
### new-omni-completion、SuperTab插件实现自动补全
使用步骤 
 
	1. 输入几个单词后，按下Tab键进行选择
	  
 **注：程序修改后，需重新运行ctags -R** 

*参考文档：[手把手教你把Vim改装成一个IDE编程环境(图文)][2]*

# IDE编程环境
除了VIM编辑器，还有许多集成开发环境可供选择。比如以下两种。 

- eclipse CDT 
- QT creator

在实践中，我在Ubuntu16.04上安装过[Intel® System Studio IoT Edition(linux版)][3]开发Intel NUC DE3815网关的C语言程序，该IDE是基于eclipse的。同时，也使用过[eclipse for ARM(windows 32版)][4]开发FS4412裸板的汇编语言程序，并未做深入研究。

# 个人观点
我认为，IDE的使用原理和**vim + Makefile + gcc**的方式一致，只是在交叉编译环境配置和可视化编程等方面有所不同。

关于vim的高级配置以及IDE的使用，待基础技能熟悉后，视情况再做深入讨论。


[1]:http://www.jianshu.com/p/bcbe916f97e1
[2]:http://blog.csdn.net/wooin/article/details/1858917
[3]:https://software.intel.com/en-us/iot/tools-ide/ide/iss-iot-edition
[4]:http://www.eclipse.org/downloads/packages/eclipse-ide-cc-developers/keplersr2