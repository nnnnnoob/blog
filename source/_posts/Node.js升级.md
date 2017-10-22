---
title: Node.js升级
date: 2017-07-15 13:34:01
categories: 日常case
tags: [node]
comments: true

---
Node.js版本更新很频繁，由0.1x版本跃升至4.x乃至目前最新的8.x版本，升级Node.js很有必要<!-- more -->

1. 查看Node版本  
```shell
$ node -v
```

2. 安装n模块  
```shell
$ npm install -g n
```

3. 查看所有node版本  
```shell
$ n ls
```

4. 升级Node  
```shell
$ n 4.4.0（版本号） # 升级到指定版本
$ n stable # 升级最新的稳定版本
```
