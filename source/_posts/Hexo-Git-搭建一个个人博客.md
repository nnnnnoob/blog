---
title: Hexo+Git 搭建一个个人博客
date: 2017-07-04 09:21:15
categories: 开源工具
tags: [Hexo,GitHub,博客]
comments: true

---
#### 使用Github Pages搭建个人网站

1. [Github Pages][1]是[GitHub][2]免费提供给开发者的一款托管个人网站的产品，首先需要创建一个GitHub账号

2. 在GitHub创建一个仓库：username/username.github.io，username是你的账户名，这是特殊的命名约定

3. 在此仓库提交一个index.html文件，网站内容是在master分支下的

4. 访问<http://username.github.io>，就能看到自己的网站了<!-- more -->

#### 绑定独立域名

1. 如果你有自己的域名的话，在之前的仓库点开设置(Settings)，找到GitHub Pages，在Custom domain添加自己的域名，保存(Save)即可

2. 在你的域名注册提供商那里配置DNS解析，获取[GitHub的IP地址][3]，添加A记录即可

#### 使用Hexo搭建博客

1. 前提  
已安装[Node.js][4]和[Git][5]

2. 安装[Hexo][6]  
```shell
$ npm install -g hexo-cli
```

3. 建站  
```shell
$ hexo init <folder>
$ cd <folder>
$ npm install
```

4. 安装 hexo-server  
```shell
$ npm install hexo-server --save
```

5. 生成静态文件  
```shell
$ hexo generate # 或者 hexo g
```

6. 启动本地服务  
```shell
$ hexo server # 或者 hexo s
```

7. 查看结果  
访问<http://localhost:4000/>，已经可以看到一篇内置的blog了，Hexo使用[Markdown][7]解析文章

#### 部署Hexo到Github Pages

1. 安装[hexo-deployer-git][8]  
```shell
$ npm install hexo-deployer-git --save
```

2. 修改配置文件[_config.xml][9]  
注：配置文件中，冒号后面一定要加空格  
```
deploy:
	type: git
	repo: git@github.com:username/username.github.io.git
	branch: master
```

3. 执行部署  
```shell
$ hexo deploy # 或者 hexo d
```

4. 使用git部署  
将我们之前创建的repo克隆到本地，将hexo generate生成public文件夹下的内容copy到本地repo，然后使用git commit提交代码，最后push到远程repo的master branch上即可

5. 查看结果  
访问<http://username.github.io>，能看到Hexo内置的blog已经发布到自己的网站了

6. 使用独立域名  
```shell
$ cd source/
$ touch CNAME
$ vim CNAME # 输入你的域名
```

#### Hexo主题Next及常用第三方服务

1. Next主题  
Hexo支持更换[多种主题][10]，本站所用主题是：[Next][11]，将喜欢的主题放在themes文件夹内，并修改_config.yml内的theme设定，即可切换主题  
```
theme: next # Hexo默认主题是landscape
```

2. 标签与分类  
确认站点配置文件以及主题配置文件里开启响应设置，新建标签和分类页面内容，在新发布的blog首部设置自定义标签和分类即可  
```
# 站点配置文件
tag_dir: tags
category_dir: categories  

# 主题配置文件
tags: /tags
categories: /categories  

# 新建source/tags/index.md
title: tags
date: 2015-10-20 06:49:50
type: "tags"
comments: false  

# 新建source/categories/index.md
title: categories
date: 2015-10-20 06:49:50
type: "categories"
comments: false  

# 新发布的blog首部设置
categories: 类别
tags: [标签1,标签2,标签3]
```

3. 网站统计[百度统计](https://tongji.baidu.com/)  
登录百度统计，定位到站点的代码获取页面，复制 hm.js? 后面那串统计脚本 id，修改主题配置文件内的字段 baidu_analytics，值设置成你的百度统计脚本id  
```
# Baidu Analytics ID
baidu_analytics: ****
```

4. 阅读次数统计[LeanCloud](https://leancloud.cn/)  
注册LeanCloud帐号，创建一个应用，在应用的配置界面创建Class（名字必须为**Counter**），由于LeanCloud升级了默认的ACL权限，如果你想避免后续因为权限的问题导致次数统计显示不正常，建议在此处选择**无限制**。在应用Key界面拿到AppID以及AppKey这两个参数，修改主题配置文件内对应的的字段即可。另外，建议开启Web安全选项，在安全中心的Web安全域名中填入我们自己的博客域名，来确保数据调用的安全  
```
# leancloud_visitors:
enable: true
app_id: ****
app_key: ****
```

5. 评论系统[HyperComments](https://www.hypercomments.com/)  
首先需要在文章首部开启评论，Next官网提供的DISQUS、多说、网易云跟帖等都已经停止服务了，只好使用HyperComments，创建一个免费应用即可，拿到WIDGET-CODE，修改主题配置文件内对应的的字段即可    
```
# 文章首部设置
comments: true  

# 主题配置文件
# Hypercomments
hypercomments_id: ****
```

6. 分享系统[jiathis](http://www.jiathis.com/)  
注册jiathis账号，获取uid后更改配置文件内对应的的字段即可  
```
jiathis:
	enable: false
	id: 2139015
```

7. 搜索服务[LocalSearch]()  
安装 hexo-generator-searchdb，编辑站点配置文件和主题配置文件  
```shell
# 安装 hexo-generator-searchdb
$ npm install hexo-generator-searchdb --save  

# 站点配置
search:
	path: search.xml
	field: post
	format: html
	limit: 10000  

# 主题配置
local_search:
	enable: true
```

8. 打赏服务  
编辑主题配置文件，添加comment和支付二维码即可  
```
# 打赏配置 可自己添加
reward_comment: 您的支持将鼓励我继续创作！
wechatpay: # 微信收款二维码
alipay: # 支付宝收款二维码
```

9. [腾讯公益404页面](http://www.ixirong.com/404.html)  
寻找丢失儿童，让大家一起关注此项公益事业！新建404.html页面，放到主题的source目录下，内容如下  
```html
<!DOCTYPE HTML>
<html>
<head>
	<meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
	<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
	<meta name="robots" content="all"/>
	<meta name="robots" content="index,follow"/>
	<link rel="stylesheet" type="text/css" href="https://qzone.qq.com/gy/404/style/404style.css">
</head>
<body>
	<script type="text/plain" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="/" homePageName="回到我的主页">
	</script>
	<script src="https://qzone.qq.com/gy/404/data.js" charset="utf-8"></script>
	<script src="https://qzone.qq.com/gy/404/page.js" charset="utf-8"></script>
</body>
</html>
```

[1]: https://pages.github.com/ "Github Pages 官网"
[2]: https://github.com/ "GitHub 官网"
[3]: https://help.github.com/articles/setting-up-an-apex-domain/ "GitHub的IP地址"
[4]: https://nodejs.org/en/ "Node 官网"
[5]: https://git-scm.com/ "Git 官网"
[6]: https://hexo.io/ "Hexo 官网"
[7]: http://www.markdown.cn/ "MarkDown 文档"
[8]: https://github.com/hexojs/hexo-deployer-git "hexo-deployer-git"
[9]: https://hexo.io/docs/configuration.html "Hexo 配置"
[10]: https://hexo.io/themes/ "Hexo 主题"
[11]: http://theme-next.iissnan.com/ "Next 官网"