---
title: 嵌入式学习笔记--前言
date: 2017-09-08 18:36:26
categories: 嵌入式
tags: [嵌入式,学习笔记,嵌入式硬件平台]
comments: true

---
# 写在前面的话
动手写博客前，我一直在思考：博客……写点啥，为了啥，有意义么。第一次写博客，难免有些紧张和忐忑。
    
我从2017年2月到9月系统的学习了嵌入式Linux的相关知识，包括嵌入式Linux应用层和底层开发等。另外，从去年9月工作以来，我在工作项目中也接触了几款嵌入式设备。因此，我希望借写博客开启嵌入式小白的学习成长之旅。<!-- more -->

对于开篇的问题，有以下几点说明： 

	1. 总结的首要目的是梳理知识点，巩固加深理解，熟练掌握开发技能。
	  
	2. 总结的内容为嵌入式Linux重点知识整理，掺杂日常实践中的问题分析。 
	
	3. 最后，博客不仅分享的是学习资料，也是笔者的学习心得，而后者是不可复制的。我想，这就是最大的意义所在。此外，若这些短文能够带给您一点帮助或启发，那将是我莫大的荣幸。


# 嵌入式平台
## [树莓派3 B][1]
- SoC： BCM2837

- CPU： **ARM Cortex-A53** 1.2GHz 四核

- **GPU**： Broadcom VideoCore IV, OpenGL ES 2.0, 1080p 30 h.264/MPEG-4 AVC 高清解码器

- 内存： 1GB

- 外设： WiFi、**蓝牙4.1**、10/100网口、USB2.0*4、microSD、HDMI、3.5mm音频插孔、CSI摄像头接口、DSI显示接口、40pin扩展GPIO


- OS： **Debian GNU/Linux**、Fedora、Arch Linux、RISC OS、Windows10、Snappy Ubuntu Core

## [JS9331开发板][2]
- SoC： Atheros AR9331

- CPU： AR9331 **MIPS 24K** 400MHz

- 内存： 64MB DDR2 SDRAM

- 外存： 8MB/16MB SPI flash

- 外设： WiFi、10/100网口\*2、USB2.0\*2、TTL/RS232串口、USB mini串口、LED\*4、key\*4、板载温度传感器、红外发送接收、5pin扩展GPIO
  
- OS： **openwrt**

## [Intel NUC DE3815TYKHE套件][3]
- CPU：**Intel x86** Atom E3815 1.4GHz 单核

- 内存：单条SO-DIMM内存插槽(最大容量8GB) DDR3

- **缓存**：512KB

- 外存：4GB eMMC闪存、2.5寸SATA硬盘位

- 外设：半长式mini PCI-E扩展插槽（无线网卡）、10/100/1000网口，USB 3.0、USB 2.0\*2、VGA、HDMI、音频插孔

- OS：**Intel IoT Gateway Software Suite**、Ubuntu 16.04 LTS、Wind River Pulsar Linux

## [IOT4412/FS4412开发板][4]
- SOC： 三星Exynos4412 

- CPU： **ARM Cortex-A9** 1.4GHz 四核

- 内存： 1GB DDR3 

- 外存： 4GB eMMC

- 外设： 20针WiFi接口、microSD、USB2.0\*2、USB OTG、10/100网口、音频插孔、串口\*2、key\*5、20PIN扩展GPIO、20针camera接口、HDMI、LVDS\_LCD\*2、RGB\_LCD

- OS： **Android4.2**、Linux-QT、Ubuntu 
 
*注：此处以IOT4412说明，FS4412与之类似，外设稍有差别*


## 附：uname -a命令结果  
	- 树莓派3 B：
		Linux raspberrypi 4.9.35-v7+ #1014 SMP Fri Jun 30 14:47:43 BST 2017 armv7l GNU/Linux 
 
	- JS991开发板：
		Linux JoySince 3.10.49 #10 Fri Jul 21 18:31:20 CST 2017 mips GNU/Linux  

	- Intel NUC DE3815TYKHE
		Linux WR-IDP-42C1 3.14.58_IDP-XT_3.1-WR7.0.0.13_idp #1 SMP PREEMPT Tue Jun 7 16:52:19 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux  

	- IOT4412/FS4412开发板
		Linux iTOP-4412 3.0.15 #3 SMP PREEMPT Thu Apr 2 18:49:01 PDT 2015 armv7l GNU/Linux
	

[1]:http://www.waveshare.net/shop/RPi3-B.htm
[2]:https://item.taobao.com/item.htm?ft=t&spm=a21m2.10192351.0.0.74c17e99LuNy2O&id=520092804695
[3]:https://www.intel.com/content/dam/support/us/en/documents/boardsandkits/DE3815TYBE_TechProdSpec.pdf
[4]:http://www.topeetboard.com/Product/iTOP4412-ss.html