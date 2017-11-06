---
title: Linux操作系统使用--3软件包管理
date: 2017-10-15 09:21:15
categories: 嵌入式
tags: [嵌入式,学习笔记,软件包管理]
comments: true

---
>本篇文章围绕Linux平台下软件包管理问题，首先介绍deb软件包和rpm软件包两种常用的软件包管理机制，重点介绍deb软件包的dpkg命令和apt原理，简述rpm软件包的rpm命令和yum工具。然后归纳在实践中使用的除上述两种软件包管理之外的方法，并结合所使用的硬件平台给出一些具体的软件包安装实例说明。 <!-- more -->


# 软件包管理机制
**定义** 
把应用程序的二进制文件、配置文档、man/info帮助页面等文件合并打包在一个文件中，使用软件包管理器操作软件包，完成获取、安装、卸载、查询等操作。
  
**主要功能** 
安装、卸载、查询、升级、校验等。

软件包管理是在工作中必做的一项任务，但Linux系统下的软件管理和Windows下软件管理截然不同，因此需要了解Linux系统软件管理的原理、方法并熟练使用。通常有两类软件包管理机制：Debian系统的deb软件包(.deb)管理和Redhat系统的rpm软件包(.rpm)管理。

关于两种软件包管理机制的**重要说明**：

1. 原理十分类似，命令有所区别，掌握其中一种，其余的有需要时再查询资料。 
2. 由Debian或者Redhat衍生出来的Linux系统必然使用与之相关的软件包管理机制。 

# deb软件包
deb软件包管理有两种方式：dpkg本地安装和apt在线安装。dpkg安装方式适用于deb软件包已经下载到本地时进行。而APT工具适合联网时安装使用。
   
## dpkg本地安装

常用的命令有以下几个： 

	- dpkg -i xxx.deb 安装软件 
	- dpkg -r xxx 移除软件 
	- dpkg -P xxx 移除软件和配置文件 
	- dpkg -l 列出已安装软件清单 
	- dpkg -s xxx 显示某个软件安装情况 

## apt在线安装

**定义** 
APT是Ubuntu Linux中功能最强大的命令行软件包管理工具，用于获取、安装、编译、卸载、查询deb软件包和检查软件包依赖关系。它是一组命令，包括apt-get、apt-cache、apt-proxy、apt-show-versions、apt-config、apt-cdrom等。其中最常用的**apt-get**。 
 
**原理** 
1.配置软件源 
各种软件包存放在软件仓库中，软件仓库置于镜像服务器中，在**/etc/apt/sources.list**软件源配置文件中列出最合适访问的镜像站点。该文件可以直接编辑修改。

在更新源或者添加源时，修改sources.list文件，需要了解其配置项的书写格式，可参考以下文档。另外，直接访问镜像站点查看各个文件夹也有助于进一步了解APT安装软件的原理。 
*参考文档： 
[《嵌入式操作系统Linux篇》3.2.1节][1] 
[debian软件源source.list文件格式说明][2] 
[apt系统中sources.list文件的解析][3]*

2.刷新软件源 
使用apt-get update命令，从/etc/apt/sources.list文件中的每一个配置项下载软件包列表，即建立索引文件，存放在**/var/lib/apt/lists**目录下。
  
3.安装软件包
使用apt-get install xxx命令，会执行以下几个步骤：

- 本地扫描软件包列表(/var/lib/apt/lists目录)找到软件包。 
- 检查依赖关系，找到支持软件运行的所有软件包。 
- 从软件源指向的镜像站点下载软件包，存放于**var/cache/apt/archives**目录。 
- 解压软件包，自动完成应用程序的安装配置。

*注：* 
*1. apt-get install执行时的打印信息就是其步骤过程* 
*2. apt的cron脚本会限制var/cache/apt/archives目录的存储空间和其中文件的存放时间*

**特点** 
相比于dpkg，APT软件包管理的两个特点是： 

1. 检查和修复软件包的依赖关系 
2. 需要使用Internet网络获取软件包

apt-get常用命令如下：

	- apt-get update 下载更新软件包列表信息
	- apt-get upgrade 软件包升级为最新版本 
	 
	- apt-get install xxx 安装软件 
	- apt-get --reinstall install xxx 重新安装软件 
	
	- apt-get check 检查是否有损坏的依赖 
	- apt-get -f install 修复软件包依赖问题 
	  
	- apt-get remove xxx 卸载软件 
	- apt-get --purge remove xxx 完全卸载软件 
	- apt-get autoremove 将不满足依赖关系的软件包自动卸载
	    
	- apt-get clean	删除缓存区所有已下载的包文件 
	- apt-get autoclean 删除缓存区老版本已下载的包文件
*注：apt-get autoclean在释放磁盘时会用到* 
  
apt-cache常用命令如下：
 
	- apt-cache show xxx 获取二进制软件包详细描述信息 
	- apt-cache search xxx 根据正则表达式检索软件包 
	- apt-cache depends xxx 查询该软件包的依赖信息 
	- apt-cache rdepends xxx 查询所有依赖该软件包的软件包 
	- apt-cache policy xxx 查询软件包安装状态 

# rpm软件包
rpm软件包管理是另外一种软件包管理机制，和deb功能十分相似。在此不再赘述，有需要时请查阅参考资料。

## rpm本地安装
rpm的功能类似于dpkg

## yum在线安装
yum功能类似于apt，也有相应的配置文件/etc/yum.conf，/etc/yum.repos.d/目录，数据库目录/var/lib/rpm/等。

*rpm和yum使用详情，参考[linux软件包管理之一（rpm包管理）][4]* 
*deb和rpm软件包管理命令对比，参考[Linux软件包管理][5]*

# 应用实例
值得注意的是，上述两种软件包管理机制适用的Linux平台为Debian和Redhat系统及它们衍生的Linux系统，而对于**嵌入式Linux**安装软件又有其各自的特殊之处。另外，基于Linux开源的思想，还有一种特别的软件安装方法，即源码安装。 

因此，在实践中，有两个方面需要考虑。第一，Linux平台有哪几种软件安装方法可供选择。第二，具体在哪种Linux平台上安装软件，需要注意什么问题。

## Linux平台下安装软件的几种方式

1. 使用dpkg、rpm等底层工具安装 
需要将软件包下载到本地后，再使用命令安装。 

2. 使用apt、yum等上层工具安装 
联网安装，自动检查依赖关系。 

3. 源代码编译安装 
第三方开源软件通用的安装方法，通常经历：下载\-\-解压\-\-./configure\-\-make\-\-make install等步骤。不同的软件以及不同的安装平台具体细节有差别，该方法可做专题研究，尤其是在嵌入式Linux平台移植，可做总结。 

4. 其他 
除了以上安装方法，有的软件提供安装脚本，直接运行即可。还有其他方法参考以下文档。 
 
*参考文档：[汇总linux下安装软件的几种方式][6]*

## 实践中各平台的操作系统
在实践中我使用了四个硬件平台，关于其详细介绍参考[嵌入式学习笔记\-\-前言][7]，在此不再赘述。
使用[cat /proc/version命令或者cat /etc/issue命令][8]查看各平台的操作系统详细信息。总结如下：
	
	- 树莓派3 B：Raspbian GNU/Linux 8
 
	- JS991开发板：openwrt
		 
	- Intel NUC DE3815TYKHE：Wind River Linux 7.0.0.13 

	- IOT4412/FS4412开发板：Linux version 3.0.15

## 实例注意事项总结
### 树莓派/Ubuntu 

**说明** 

1. 树莓派和Ubuntu都基于Debian系统，因此安装软件首选apt工具。 
2. 首先尝试apt-cache search xxx 和 apt-get install xxx。 
3. 若提供该软件的软件源，则需要使用apt-key命令导入GPG密钥；配置软件源文件，下载至/etc/apt/sources.list.d/或手动创建编辑；再apt-get update和 apt-get install xxx。 
4. 对于ppa个人软件包，使用add-apt-repository命令添加软件源，再apt-get update和 apt-get install xxx。 
5. 使用git clone、wget等方法下载安装包，使用提供的脚本安装或者编译源码安装。

**实践example** 

- Ubuntu下安装mosquitto服务器进行mqtt通信实验 
- Ubuntu下安装Tomcat web应用服务器进行Java后台开发实验 
- Ubuntu下安装docker，[Intel® System Studio IoT Edition][9]进行Intel网关开发 
- Ubuntu下安装gsoap进行onvif协议开发
- 树莓派下安装FFmpeg进行视频推流实验 
- 树莓派下安装node、npm，利用[AWS IOT JavaScript SDK][10]进行树莓派连接AWS云实验 
- 树莓派下安装[Azure IOT][11]开发包进行树莓派连接Azure云实验 

*注：有的软件安装涉及环境变量配置，和Windows下类似*
  
### JS991开发板
**说明** 

1. openwrt系统下软件包管理工具为[opkg][12],通常使用opkg install xxx.ipk。 
2. 如果安装失败，提示缺少依赖的库，可以先使用opkg install安装该库或者找到openwrt系统中该动态链接库，将其复制到开发板的/usr/lib下，再安装相关软件。 
3. ipk软件包可以来源于openwrt下载官网，也可以自己制作并在内核中编译生成，具体制作方法另起专题研究。
4. 支持LuCi进行页面可视化安装软件。
  
*参考文档：[openwrt安装软件的两个方法][13]* 

**实践example**

- openwrt下移植opencv，进行视频图像处理实验。 

### Intel NUC DE3815TYKHE
**说明** 

1. wind river Linux系统作为Intel IoT Gateway Software Suite的一部分提供，此处使用的是免费版本Wind River Linux 7.0.0.13。 
2. [Wind River Linux 7.0.0.13][14]系统下的软件包后缀为rpm，软件包管理工具为smart update、smart install、smart channel \-\-add等。可能类似rpm软件包管理，具体需深入学习该系统。 
3. 若提供软件源，需要使用rpm \-\-import命令导入GPG密钥，然后使用smart channel \-\-add添加软件源，再使用smart update和smart install。 
4. 添加软件源镜像所能访问的文件夹需要和操作系统版本保持一致。 
5. 支持页面添加源和软件安装。 
 
**实践example**

- wind river Linux下安装AWS IOT SDK,进行[Intel nuc3815连接AWS云实验][15]
- wind river Linux下安装paho.mqtt.c,进行mqtt客户端通信实验。
- wind river Linux下安装OpenSSL，用于mqtt安全通信。


>声明：文章中涉及的参考文献并不是随意引用，均是经过笔者阅读大量资料筛选后所得，和本文有密切关系，可放心阅读。

[1]: http://www.ryjiaoyu.com/book/details/4461
[2]: http://blog.csdn.net/nodeathphoenix/article/details/17958379
[3]: http://blog.sina.com.cn/s/blog_406127500101dfv1.html
[4]: http://fanqie.blog.51cto.com/9382669/1728040
[5]: http://www.linuxidc.com/Linux/2015-12/126456.htm
[6]: http://www.360doc.com/content/16/1006/11/29770038_596138348.shtml
[7]: http://www.youbo.website/2017/09/08/%E5%B5%8C%E5%85%A5%E5%BC%8FLinux%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0--%E5%89%8D%E8%A8%80/
[8]: http://www.cnblogs.com/lanxuezaipiao/archive/2012/10/22/2732857.html
[9]: https://software.intel.com/en-us/intel-system-studio-iot-edition-guide-for-c-installing-on-linux
[10]: https://www.npmjs.com/package/aws-iot-device-sdk
[11]: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-raspberry-pi-kit-c-get-started
[12]: https://wiki.openwrt.org/zh-cn/doc/techref/opkg
[13]: http://www.openwrtdl.com/wordpress/openwrt-install-software
[14]: https://knowledge.windriver.com/en-us/000_Products/000/010/000
[15]: https://software.intel.com/en-us/gateway-getting-started-software-suite-connecting-to-amazon-web-services