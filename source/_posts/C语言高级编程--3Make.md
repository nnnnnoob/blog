---
title: C语言高级编程--3Make
date: 2017-11-23 09:21:15
categories: 嵌入式
tags: [嵌入式,学习笔记,make,Makefile,autotools,cmake]
comments: true

---
> 本篇文章围绕GNU make介绍了make工程管理器相关知识，包括Makefile基本结构、变量、规则的使用，以及一些相关问题：伪目标、文件搜索、嵌套执行make、Makefile函数的使用。最后结合实践情况，对一些Makefile文件做了简要分析，并介绍了autotools和cmake两种自动生成Makefile的工具。<!-- more -->


# make简介
- make工程管理器，它是一个自动编译管理器，“自动”是指它能够根据文件时间戳自动发现更新过的文件，只编译改动的代码文件，而不是全部编译，从而减少工作量。make可以提高项目开发和维护的工作效率。
- Makefile是make读入的唯一配置文件。
- 可以理解为makefile里面是大量的gcc命令。使用Makefile可以减少多个文件编译时一遍一遍输入gcc命令。只不过Makefile有其自己的语法格式。
- 这里是总结GNU make的知识。而其他厂商的make与之类似。

# Makefile基本结构
使用make工程管理器，也就是编写Makefile文件。Makefile文件由三部分组成：（1）目标体：通常是可执行文件、目标文件、标签。（2）创建目标体所依赖的文件。（3）创建每个目标体需要运行的命令。

Makefile格式：
```
target:dependency_files
<tab> command
<tab> ……
```
说明：

1. Makefile的工作原理：如果dependency_files比target新或者target不存在，那么就会执行command指令。
2.  目标和依赖是Makefile中的主要构成。而目标和依赖的关系是通过规则表达，即命令。
3. 一个Makefile可以定义多个目标，调用make命令时需指明目标，如果未指明时，默认生成第一个目标。
4. 如果依赖也是目标，那么make会按从左到右的顺序先构建规则中依赖的目标，再构建目标。Makefile只检查依赖关系，如果依赖的文件找不到则会退出。
5. 注意命令前tab键的使用。Makefile以tab键区分是否是命令，不能以多个空格代替。
6. command不一定是编译链接等命令，可以是任意可以运行的命令以及make定义的函数。
7. 一个规则可能包含多条命令。一个规则也可以包含多个目标。
8. 在Makefile文件中除了有规则、变量定义还有文件指示和注释。文件指示包括：在一个Makefile中包含另一个Makefile；指定Makefile的有效部分，即条件判断；定义一个多行的命令。而注释是以#开头。
9. 包含其他Makefile，可以使用include  filename，其中filename可以包含路径和通配符。
10. 使用cc -MM xxx.c命令可以输出文件的依赖关系，并生成一条依赖语句。

# 变量使用
- 在Makefile中使用变量的目的就是代替一个文本字符串，作用类似于宏。比如系列文件的名字、传递给编译器的参数、运行的程序、查找源代码的目录。使用变量可以增加Makefile的可维护性。
- 在Makefile文件中定义的变量相当于全局变量，如果要定义局部变量，可以定义在目标之后或者模式（%）之后，则变量会只作用于由这个目标引发的所有规则。

## 定义变量的两种方式

- 递归展开方式 
```
VAR=var
```
 一次性将内嵌的变量全部展开，但不能在变量后追加内容，可能导致无穷循环。
 
- 简单扩展方式 
```
VAR：=var
```
  在定义处展开，且只展开一次。前面的变量不能使用后面的变量，只能使用前面定义好的变量。

## 使用变量
```
$(VAR)
```

使用说明:

1. 变量大小写敏感。
2. 推荐使用小写字母作为变量名，预留大写字母作为控制隐式规则参数或用户重载命令选项参数的变量名。
3. 实践中常会用到+=，为定义的变量添加新的值。
4. 变量的替换
(1) $(var:a=b) 将var变量中以a结尾的字符串替换为b。
(2) 使用静态模式 $(var:%a=%b)。
5. override指示符的作用：如果变量是通过命令行参数设置，要在Makefile中设置其值，需要使用override。
6. 定义空格可以写为：
```
empty:=
space:=$(empty) $(empty)
```
## 变量的种类

- 自定义变量
- 预定义变量
- 自动变量 
	-  $@目标文件完整名称，在模式规则中， 可以匹配目标中模式定义的集合。
	-  $^所有不重复的目标依赖文件。
	-  $<第一个依赖文件名称，在模式规则中，是将符合模式的文件挨个取出。
- 环境变量 
make在启动时会自动读取系统当前已经定义的环境变量，并创建与之相同的变量，若在Makefile中定义了相同名称的变量，则会覆盖同名的环境变量。 类似于全局变量和局部变量的关系。最好不要使用环境变量MAKEFILES,否则会影响使用make。

# 规则说明
- Makefile的规则包括目标体、依赖文件和其间的命令语句。这称为显式规则，即显示的说明了如何生成一个或多个目标文件。除此之外，还有隐式规则和模式规则。
- 规则包含两个部分：一个是依赖关系，一个是生成目标的方法。Makefile中只应该有一个最终目标，第一条规则中的目标被确立为最终的目标，所以顺序很重要。

## 隐式规则
- 隐式规则告诉make如何按照约定俗成的方法完成任务，只需把目标文件列出即可，不需要指定编译的具体细节。
- 使用隐含规则生成目标，make会自动推导这个目标的规则和命令。
- 在隐含规则库中，每一条规则都在其库中有顺序，位置靠前的规则容易被使用，为了避免依赖无效，则必要时需要显示写出执行的命令，不能省略。

常见的隐式规则有：
```
.c变成.o	使用$(CC) -c $(CPPFLAGS) $(CFLAGS)
.cc变成.o	使用$(CXX) -c $(CPPFLAGS) $(CXXFLAGS)
.o生成目标	使用$(CC) $(LDFLAGS) xxx.o $(LOADLIBES) $(LDLIBS)
```
说明：

1. 其余隐式规则详见《跟我一起写Makefile:隐含规则》。
2. 隐式规则中使用的是系统设定的变量，用户可以在Makefile中修改这些变量的值，只要设置了其值，那么就会对隐式规则起作用。
3. 关于隐含规则链，一个目标可能被多条隐含规则作用。make会尽一切可能生成目标，其中可能生成中间文件，这种方式生成的中间文件在生成目标后会使用“rm -f”删除。

## 模式规则
- 模式规则用来定义对多个文件使用相同处理规则。隐式规则仅能使用make默认的变量进行操作，但模式规则可以引入用户自定义变量，简化Makefile的书写。
- 模式规则在目标定义中需加上"%"符号，"%"表示一个或多个任意字符。目标必须是这种形式，否则成为一般的规则。
- 依赖中也可以使用“%”符号，但是其值取决于目标。

使用示例：
```
%.o:%.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

**补充**：静态模式，可以容易的定义多目标的规则。其定义和模式规则类似，格式如下：
```
target:target-pattern:prereq-pattern
<tab>commands
```
其中target为一系列目标，target-pattern是目标集模式，prereq-pattern是目标的依赖模式。
使用示例：
```
objects = foo.o bar.o
all: $(objects)
$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
```

# 相关问题
## 伪目标
使用.PHONY关键词定义伪目标，作用是防止文件夹中有和目标相同名字的文件，执行make指令不产生预期的动作。例如clean目标，可以认为是一个标签，不依赖任何文件。因此可以在Makefile中定义一些和编译无关的命令，比如程序的打包、备份、删除等。

使用示例：
```
.PHONY:clean
clean:
	rm xxx *.o 
```
说明：

1. 通常clean规则写在文件最后。需要显示使用make clean执行相应功能。
2. 这里使用了通配符“\*”，make支持“\*"、"?"、"~"三种通配符。
3. 如果要一次生成多个可执行文件，可以使用all伪目标，用伪目标作为默认目标。
4. 伪目标也可以作为依赖，用于完成不同的功能，详见《跟我一起写Makefile:书写规则》。

## 文件搜寻
使用VPATH完成文件搜索。如果Makefile文档中定义了VPATH变量，那么make会在当前目录找不到的情况下，到指定的目录中寻找文件。

使用示例：
```
VPATH=src:../header
```
说明：

1. make会按照这个顺序进行搜索，多个路径由冒号分割。
2. 还可以使用vpath(小写字母)关键词定义，详细使用方法详见《跟我一起写Makefile:书写规则》。

## 命令的书写
这里总结几条书写命令时的注意事项：

1. make的命令默认使用/bin/sh解释执行。
2. 如果在使用make时不希望打印命令，则在Makefile中命令前加上@。
3. 如果要执行两条命令，第一句的命令结果要应用于下一条命令，必须用分号分割，而不能写在两行。
4. 在Makefile中可以为相同的命令序列定义一个变量，注意不可以和变量重名。使用方法和变量类似，执行命令时，其包裹的每条命令都会执行。
使用示例：
```
define two-lines 
	echo foo
	echo $(bar)
endef
```

## 嵌套执行make
- 在大的工程中，不同模块或功能的源文件放在不同的目录中，每个目录都可以书写一个该目录下的Makefile。最外层的Makefile称为总控Makefile。
- 如果要传递变量到下级Makefile中，使用export xxx，如果要传递所有，使用一个export即可。其具体使用详见《跟我一起写Makefile:书写命令》。
- 在实践中，通常有很多源码，每个文件夹都有Makefile,上级目录中的Makefile会逐级调用下层的Makefile。具体情况需结合实例分析。
- 使用make -C指定进入下层Makefile时，-w参数自动打开，则可以在make执行时看到进入哪个目录，离开哪个目录。

使用示例:
```
set -e; for d in $(DIRS); do $(MAKE) -C $${d}; done
```

## 使用条件判断
使用条件判断，可以让make根据不同的情况选择不同的分支执行。格式如下：
```
ifeq (arg1,arg2) 或者ifneq (arg1,arg2) 或者 ifdef xxx 或者ifndef xxx
……
else
……
endif
```
说明：

1. arg可以是make函数。
2. ifdef只是判断变量是否为空，非空为真，空为假。

## Makefile函数
在Makefile中使用函数处理变量，可以增加其灵活性。函数的返回值可以作为变量使用。包括字符串处理函数，文件名操作函数，foreach，if，call，shell函数等，具体使用详见《跟我一起写Makefile:使用函数》。

格式如下：
```
$(function arguments)
```
函数名和参数用空格隔开，变量之间用逗号隔开。

常用的函数总结如下：
```
addprefix 给字符串中每一个子串加上一个前缀。
$(addprefix prefix,names)

filter 从字符串中根据模式得到满足模式的字符串，返回匹配的。
$(filter pattern,text)

filter-out 从字符串中根据模式滤出某些字符串，返回不匹配的。
$(filter-out pattern,text)

wildcard 通配符，相当于*，pattern举例：*.c
$(wildcard pattern)

patsubst 模式字符串替换,相当于变量替换的功能。
$(patsubst pattern,replacement,text)

shell函数
执行shell命令，和`xxx`是同样的功能。
```

# make 使用
## make命令

- make命令执行后会有三个退出码。0表示成功执行，1表示运行时出现任何错误。2表示某些目标不需要更新，make需要使用-q选项。
- make最终目标是Makefile中的第一个目标。可以指定make的目标，只需在make命令后跟上目标的名字即可。目标可以是真实的目标，也可以是伪目标。
- 通常开源软件发布的Makefile中都包括编译、安装、打包等功能。例如有all、clean、install、tar、dist、test等目标。具体要结合具体的Makefile分析。
- 因此，可以使用make xxx，make clean，make install等完成不同的功能，如果只写make,则默认建立Makefile中第一个目标。

## make 常用参数

```
-C dir	读入指定目录下的Makefile。
-f file	读入当前目录下file文件作为Makefile。
-n		模拟执行，打印要执行的命令，但不执行。可以用于调试Makefile，查看其执行顺序。
-j		同时运行命令的个数，-j2表示以两个线程编译。
-i		忽略所有命令执行错误，也可以在Makefile中command前加上短横线。
```
更多的参数详见《跟我一起写Makefile：make运行》。

# 应用实例
- 在实践中，通常会用到第三方的开源库或者SDK包，因此涉及第三方库的移植，若是源代码，那么需要修改其自带的Makefile，使其适合目标平台，再进行配置编译工作。该部分内容另作讨论研究。这里只对使用第三方库过程中的Makefile做简单分析。
- 此外，在开发者自己编写模块或者软件包时，可以采用系统内核规定的方式，将软件包作为内核的一部分进行编译。也可以放在内核之外，使用内核的Makefile文件编译为模块，使用时再进行动态加载。这里也做简要分析。
- 最后，实践中也经常使用工具来自动生成Makefile,例如autotools和cmake工具，开发者只需按照规定填充源文件的相关信息。

## 第三方开源库或SDK使用中的Makefile分析

### openwrt使用opencv库Makefile分析
在js9331开发板上做图像处理，需要openwrt移植opencv库。具体移植的方法在后续第三方库移植专题中再做深入研究。这里只对其测试代码的Makefile做一分析。

Makefile说明：

1. Makefile代码详见[gist][25]。
2. 使用命令时，可以添加命令行参数，例如export prefix=xxx; make install。
3. opencv的应用程序一般使用c++编写，如果移植到openwrt平台，涉及到c和c++程序的混合编译，使用注意事项详见[《c语言高级编程-2搭建交叉编译环境》][26]。

*参考文档：*
*[用OpenWrt package方式编译OpenCV][1]*
*[ openwrt-packages/wrtnode/opencv-test/src/Makefile][2]*

### ubuntu使用海康威视SDK包Makefile分析
海康威视摄像头提供linux版本的SDK包，供开发者进行二次开发。其中涉及到Makefile文档，对其进行简要分析。

Makefile说明：

1. Makefile代码详见[gist][27]。
2. gcc选项-W……，用于不同错误的警告提示。
3. Makefile函数的使用，比如subst，foreach，wildcard，dir，notdir，patsubst，addprefix等。
4. VPATH的使用，用于文件搜索。
5. 对于动态链接库，使用-Wl,-rpath=……选项，表示运行时记住库的路径。
6. gcc选项，-pipe，用于优化处理。

*参考文档：*
*[提领类型双关的指针将破坏重叠规则——strict-aliasing][3]*
*[gcc中的-Wl,rpath=<your_lib_dir>选项][4]*
*[《gcc五分钟系列》第十节：编译期优化选项（一）——pipe ][5]*
*[海康威视设备网络SDK_V5.2.7.4(for Linux64)][6]*

### openwrt上onvif开发Makefile分析
openwrt平台上进行onvif协议开发和在其他linux平台上类似，只是编译器选为mips-openwrt-linux-gcc。onvif协议客户端开发，使用gSoap开源软件中的WSDL2h和SOAPcpp2工具生成onvif.h和相关的.c源文件进行应用程序开发。具体原理另作研究讨论。

Makefile说明：

1. Makefile代码详见[gist][28]。
2. 使用-DDEBUG选项，可以打印onvif通信日志。
3. 使用strip命令，可以对可执行文件瘦身，减小大小，对于嵌入式开发十分必要。

*参考文档：*
*[onvif开发之设备发现功能的实现][7]*
*[Onvif开发之Linux下gsoap的使用及移植][8]*

## 编译自己的模块或软件
### openwrt编写ipk包Makefile分析
openwrt软件安装包一般为ipk后缀的文件，它提供了一种让用户自己生成ipk包的方法。即：

1. 在openwrt源码中，package目录下，新建文件夹A，其中创建Makefile和src文件夹。src文件夹中编写.c、.h源文件和编译的Makefile。或者没有源码文件，在A文件夹下的Makefile中写明获取软件源码的网址。
2. 在文件夹A下的Makefile是一种编写模板，其提供了该软件包的下载、编译、安装的方法。
3. 在步骤2编写完成的Makefile包含了该软件在make menuconfig配置下的位置和信息，因此在openwrt主目录下make menuconfig选中要编译的软件。
4. 在openwrt主目录下运行make V=s。完成后在/openwrt/bin/ar71xx/packages/base目录下可以找到对应的ipk软件包，复制到开发板进行安装即可。

Makefile说明：

1. Makefile代码详见[gist][29]，这里选取两个范例，一个是openwrt移植opencv使用该方法，另一个是普通的ipk软件包生成测试。具体含义参见参考文档。
2. makefile函数的使用，eval和call函数。

*参考文档:*
*[用OpenWrt package方式编译OpenCV][1]*
*[Openwrt学习之路-(5-Openwrt package Makefile)][9]*
*[ openwrt简单ipk生成及Makefile解释 ][10]*

### iTOP4412/FS4412编写LED灯驱动Makefile
linux驱动编译时，多采用动态编译，即驱动的源代码不放在内核中，因此其Makefile编写有一定讲究。在此简略分析。关于驱动的内容详见后续总结。

Makefile说明：

1. Makefile代码详见[gist][30]。
2. ifeq的使用，执行不同的命令。
3. make -C指明使用内核Makefile的路径。

*参考文档：*
*[Linux 驱动开发之内核模块开发 （二）—— 内核模块编译 Makefile 入门][11]*

## 自动生成Makefile的工具
对于大型的项目，如果有工具可以自动生成Makefile，那么将提高开发效率。
### GNU autotools
使用autotools制作Makefile步骤如下：

1. 运行命令autoscan。
2. 修改configure.scan文件，并重命名为configure.ac。
3. 运行命令aclocal。
4. 运行命令autoconf。
5. 运行命令autoheader。
6. 创建文件Makefile.am，并配置内容。
7. 运行命令automake --add-missing。期间缺少的文件自己创建。
8. 运行./configure命令。
9. 运行make命令，得到可执行文件。

说明：

1. make install、make clean、make disk等命令可以使用。
2. 在confiure.ac Makefile.am文件中填写源文件相关的内容，不需要填写依赖关系等，可以自动生成。

*参考文档：*
*[Linux c 开发 - Autotools使用详细解读][12]*

### cmake
- GNU autotools 是linux系统下常用的自动生成Makefile的工具，而[cmake][13]是一款跨平台的安装编译工具。不仅可以生成Makefile，还可以生成Windows visual C++的projects等。
- 在实践项目中，涉及一些开源软件的使用，比如[opencv][17]、[mosquitto][18]、[paho.mqtt.embedded-c][19]、[Azure iot-hub-c-raspberrypi-client-app][20]等。在这些源码中，包含了许多CMakeLists.txt文件。关于如何在自己的程序中使用这些第三方的开源库，在第三方库移植专题中总结。这里只对cmake做一简单介绍，涉及两个方面。
	- 如何编写CMakeLists.txt文件，即理解其语法。常用的语法如下：
			projet(projectname) 项目名称
			add_executable(exename source) 生成可执行文件
			set(var value) 设置变量
			add_subdirectory(source_dir) 增加源代码目录
			add_library(libname source ) 生成链接库
			target_link_libraries() 添加依赖的链接库
			aux_source_directory(. var) 查找当前目录下源文件并将其保存到var变量中
	- 如何使用cmake命令。
			cmake PATH PATH是CMakeLists.txt所在的目录
			cmake . 在当前目录下运行cmake
	

注意：

1. CMakeLists.txt中命令不区分大小写。
2. 使用变量时，使用花括号。

*参考文档：*
*[为什么用CMake][14]*
*[大型程序管理神器之CMake][15]*
*[CMake 入门实战][16]*
*[cmake 命令行][22]*
*[CMake Tutorial][23]*
*[CMake交叉编译配置][24]*

# 个人观点
1. 本篇文章的Makefile主要参考文档为[跟我一起写makefile][21]。
2. 写Makefile时，首先是思考需要表达什么样的依赖关系。而不是先思考如何写规则命令。
3. 对于makefile的掌握程度，要会写简单的makefile，用于自己项目管理。要会读懂别人的makefile，要会根据需要做相应修改。对于一些细节知识仅作了解，理解即可，真正使用时再做分析。
4. 在实践中，由于项目并不大，因此一般自己编写Makefile文件，可使用一份Makefile模板，进行细微修改即可，而不是用cmake或autotools工具。autotools目前只做了解。
5. 目前cmake工具的使用主要用在开源库的使用中，因此需要理解其含义和使用方法。待对cmake有足够了解后，可以对开源库的CMakeLists.txt根据自己需要进行修改。

[1]: http://wiki.wrtnode.com/index.php?title=OpenWrt_package_method/zh-cn
[2]: https://github.com/WRTnode/openwrt-packages/blob/master/wrtnode/opencv-test/src/Makefile
[3]:http://blog.csdn.net/bytxl/article/details/8091120
[4]:https://www.cnblogs.com/bourneli/archive/2012/04/27/2474103.html
[5]:http://blog.51cto.com/elephantliu/684257
[6]: http://www.hikvision.com/cn/download_more_403.html#prettyPhoto
[7]: http://blog.csdn.net/love_xjhu/article/details/11821037
[8]: http://blog.csdn.net/xuerongdeng/article/details/17927723
[9]: http://www.jianshu.com/p/3320deb24335
[10]: http://blog.csdn.net/sg656720274/article/details/54582620
[11]: http://blog.csdn.net/zqixiao_09/article/details/50838043
[12]:http://blog.csdn.net/initphp/article/details/43705765
[13]: https://baike.baidu.com/item/cmake/7138032?fr=aladdin
[14]: http://blog.csdn.net/wdkirchhoff/article/details/43486469
[15]: http://blog.csdn.net/farsight2009/article/details/60141906
[16]: http://www.hahack.com/codes/cmake/
[17]: https://github.com/opencv/opencv
[18]: https://github.com/eclipse/mosquitto
[19]: https://github.com/eclipse/paho.mqtt.embedded-c
[20]: https://github.com/Azure-Samples/iot-hub-c-raspberrypi-client-app
[21]: http://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile
[22]: http://blog.csdn.net/darkdong/article/details/6102104
[23]: https://www.jianshu.com/p/bbf68f9ddffa
[24]: https://www.cnblogs.com/rickyk/p/3875334.html
[25]: https://gist.github.com/youxiaobo/b168b82d0192b90c7d54ab1b92956525
[26]: http://www.youbo.website/2017/11/17/C%E8%AF%AD%E8%A8%80%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B--2%E6%90%AD%E5%BB%BA%E5%B5%8C%E5%85%A5%E5%BC%8FLinux%E7%BC%96%E7%A8%8B%E7%8E%AF%E5%A2%83/
[27]: https://gist.github.com/youxiaobo/891893b9e51c1ff1648ada032488548a
[28]: https://gist.github.com/youxiaobo/0289a2ee583b3f344aac434a5b569513
[29]: https://gist.github.com/youxiaobo/ed8d674a71255bf495139738ba717f25
[30]: https://gist.github.com/youxiaobo/50ba2fe2af98a91d87b31d1955480344
