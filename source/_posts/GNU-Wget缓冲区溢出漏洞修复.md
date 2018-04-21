---
title: GNU-Wget缓冲区溢出漏洞修复
date: 2018-04-21 11:41:20
categories: 日常case
tags: [漏洞]
comments: true

---
收到阿里云漏洞提醒，RHSA-2017:3075: wget security update，记录一下解决过程。<!-- more -->

### 漏洞说明
漏洞编号：CVE-2017-13089、CVE-2017-13090  
漏洞名称：wget缓冲区溢出漏洞  
涉及版本：1.19.2之前的版本  
等级：高危

### 解决方法
#### 下载新版Wget
http://ftp.gnu.org/gnu/wget
#### 编译安装新版
```shell
wget http://ftp.gnu.org/gnu/wget/wget-1.19.2.tar.gz
tar -zxf wget-1.19.2.tar.gz
cd wget-1.19.2
./configure --prefix=/usr --sysconfdir=/etc --with-ssl=openssl
make
make install
```
#### 安装依赖包
configure过程中可能会遇到No package 'openssl' found，需要安装openssl
```shell
yum list|grep openssl
yum install openssl-devel.x86_64
```
#### 查看版本并验证
```shell
wget -V
```
