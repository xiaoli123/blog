title: 使用hexo+github搭建静态博客
date: 2015-10-04 11:01:31
tags:
- hexo
categories: [hexo]
---
# 认识hexo
&#8195;&#8195;hexo是一款基于Node.js的静态博客框架，可以生成静态文件并且一键部署到github pages上，并且他可以使用markdown来编写文章，十分简单。

他有许多优点：
* 快速：hexo基于Node.js，支持多进程，几百篇文章可以秒生成。
* 写文章流程：支持GitHub Flavored Markdown和所有Octopress的插件。
* 扩展性：Hexo支持EJS、Swig和Stylus。通过插件支持Haml、Jade和Less。

*这是一篇用hexo在github上面搭建博客的教程*

***

# 安装：
## 系统环境：
* 系统： win10 64bit
* 编译器：[sublime text 3](http://www.sublimetext.com/)（可以看这篇文章）
* 编码： 把文本编码确定成 UTF-8

## Git
1. 可以选择安装GitHub for Windows 或者 git 。
2. 注册github，选择Repositories，然后new新建一个仓库，建好仓库之后选择settings，到GitHub Pages那一栏，选择Launch automatic page generator，接下去就是确定。
3. 然后就是把仓库clone下来，关于怎么clone和使用git可以参考这篇文字。

## nodejs
下载安装[nodejs](https://nodejs.org/)，nodejs安装后会自动带有npm包，如果嫌外网下npm下载包文件比较慢，可以使用淘宝的[cnpm](http://npm.taobao.org/)镜像。
nodejs安装好之后可以打开命令行（win+r，输入cmd，确定），
输入 node -v 查看nodejs版本：
``` bash
$ node -v
$ v0.12.7 ##版本号
```
输入 npm -v 查看npm版本：
``` bash
$ npm -v
$ v2.11.3 ##版本号
```
到这里说明nodejs安装成功了。

## hexo
打开命令行，我这里安装的是3.x版本（后面可能会有个小坑）
``` bash
$ npm install hexo-cli -g -d
```
查看hexo的版本号：
``` bash
$ hexo -v
```
进入到我们clone的那个文件夹，执行命令
``` bash
$ hexo init
```
接着安装依赖包：
``` bash
$ npm install
```
接着可以启动服务测试一下:
``` bash
$ hexo s
```
用浏览器打开`http://localhost:4000/`或者`http://127.0.0.1:4000/`就可以看到页面了。

***
# Hexo的使用
## 目录和文件
```
.
├── .deploy       #需要部署的文件
├── node_modules  #Hexo插件
├── public        #生成的静态网页文件
├── scaffolds     #模板
├── source        #博客正文和其他源文件, 404 、favicon 、CNAME
|   ├── _drafts   #草稿
|   └── _posts    #文章
├── themes        #主题
├── _config.yml   #全局配置文件
└── package.json
```
（有些文件是在部署时才会产生的）
## 全局站点配置文件_config.yml
（yml语法严格，注意冒号后面必须加空格）
```
# Site #站点信息
title: Hexo #博客标题
subtitle: 啦啦啦 #副标题
description: 哈哈哈 #描述
author: John Doe #作者
language: zh-Hans #语言
timezone: Asia/Shanghai #时区

# URL #链接格式
url: http://yoursite.com #博客网址
root: / #根目录
permalink: :year/:month/:day/:title/ #文章的链接格式
permalink_defaults:

# Directory #目录
source_dir: source #源文件
public_dir: public #生成的文件
tag_dir: tags #标签文件夹
archive_dir: archives #归档文件夹
category_dir: categories #分类文件夹
code_dir: downloads/code #下载文件
i18n_dir: :lang #国际化
skip_render:

# Writing #写作
new_post_name: :title.md #文章标题
default_layout: post #模板
titlecase: false #标题是否换成小写
external_link: true #是否在新页面打开链接
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:

# Category & Tag #分类和标签
default_category: uncategorized
category_map:
tag_map:

# Date #时间日期格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination #分页
per_page: 10 #每页文章数
pagination_dir: page

# Extensions #插件和主题
## Plugins: http://hexo.io/plugins/
## Themes: http://hexo.io/themes/
theme: landscape

# Deployment #部署配置
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
```
## 站点的_config.yml 配置
### 站点建立时间
例如 `` © 2015 - 2016``
在文件添加：
```
since: 2015
```
### 头像设置
/images/avatar.jpg //头像图片放置在主题的 source/images/
在文件添加：
```
avatar: /images/avatar.png
```
### 侧边栏社交链接
增加 social
```
# Social links
social:
  GitHub: https://github.com/tongchengqiu
  Twitter: https://twitter.com/tongchengqiu
  Zhihu: http://www.zhihu.com/people/tongchengqiu
```
## 友情链接
### 标题
links_title: 友情链接
### 链接
links:
  Hexo: http://hexo.io/
  Lmintlcx: http://blog.lmintlcx.com/
## 命令的使用
```
hexo help #查看帮助
hexo init #初始化目录
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成网页, 可以在public目录查看整个网站的文件
hexo server #本地预览,'Ctrl+C'关闭
hexo deploy #部署.deploy目录
hexo clean #清除缓存, 建议每次执行命令前先清理缓存
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```
## 写文章
```
hexo new "文章标题"
```
然后到_post 目录下打开 文章标题.md
```
title: 文章标题
date: 2015-10-3 11:11:11
tags:
- 标签1
- 标签2
- 标签3
categories: [分类1,分类2,分类3]
---
正文, 使用 Markdown 语法书写
```
编辑后可以先预览
```
hexo clean
hexo s
```
然后打开浏览器预览
## 部署上传更新
以github为例，打开站点根目录下的_config.yml文件，找到deploy部分，更改配置
```
deploy:
  type: git
  repository: git@github.com:test/test.git #在仓库找到该仓库的ssh
  branch: github-pages #分支，这里为github-pages
```
然后就可以部署了，在文件根目录执行命令行
```bash
$ hexo clean
$ hexo deploy
```
这里有可能会报个错误``ERROR Deployer not found : git``
这里需要安装一个包
```bash
$ npm install hexo-deployer-git --save
```
然后就可以继续执行``deploy``了，第一次部署可能会有点慢，可以耐心等候哦。
成功之后可以继续打开仓库的settings，在GitHub Pages会看到 Your site is published at http://qiutongcheng.com.（如果还没有绑定域名就会是github给你分配的url哦）
就可以打开链接看看咯~
***

# 绑定域名
当然我们的个人博客当然希望绑定自己的一个个性域名咯~

## 注册购买顶级域名
国内购买可以上[万网](http://wanwang.aliyun.com/)；
嫌国内不好可以用[godaddy](https://www.godaddy.com/)。

## DNS设置
在购买域名的后台管理设置DNS
主机记录@, 类型A, 记录值192.30.252.153
主机记录www, 类型A, 记录值192.30.252.153

## 在github绑定域名
在source文件夹下面新建 CNAME（不要后缀），内容输入要绑定的域名，比如
``qiutongcheng.com``或者``www.qiutongcheng.com``。

## 绑定子域名
比如要绑定 ``blog.qiutongcheng.com`` ：
吧CNAME内容改成 ``blog.qiutongcheng.com``
DNS设置里面添加 主机记录有blog，类型 CNAME ，记录值 ``blog.qiutongcheng.com``
***

# 主题
hexo的[(主题列表)](https://github.com/hexojs/hexo/wiki/Themes)
我使用的是 NexT 主题，这里一NexT主题为例剖析一下吧。
## 下载与使用
执行命令行
```bash
$ git clone https://github.com/iissnan/hexo-theme-next themes/next #主题的地址
```
也可以下载zip文件，然后手动解压到themes目录下
配置全局站点文件``_config.yml``
```
themes: next #主题文件夹名
```
## 主题的目录结构
```
.
├── languages          #国际化
|   ├── default.yml    #默认
|   └── zh-CN.yml      #中文
├── layout             #布局
|   ├── _partial       #局部的布局
|   └── _widget        #小挂件的布局
├── script             #js脚本
├── source             #源代码文件
|   ├── css            #CSS
|   |   ├── _base      #基础CSS
|   |   ├── _partial   #局部CSS
|   |   ├── fonts      #字体
|   |   ├── images     #图片
|   |   └── style.styl #style.css
|   ├── fancybox       #fancybox
|   └── js             #js
├── _config.yml        #主题配置文件
└── README.md          #主题介绍
```
## 主题配置文件_config.yml文件
打开主题目录下的_config.yml文件
```
menu: #菜单
  home: / #首页
  archives: /archives #归档
  #commonweal: /404.html #404页面
  #tags: /tags #标签
  #categories: /categories #分类
  about: /about #关于

# 小图标
favicon: /favicon.ico

# 默认关键词
keywords: "~"

# 留空使用默认的, false 禁用, 也可以写指定的地址
rss:

# Icon fonts
# default | linecons | fifty-shades | feather
icon_font: default

# 代码高亮主题 https://github.com/chriskempson/tomorrow-theme
# normal | night | night eighties | night blue | night bright
highlight_theme: normal

# MathJax Support #数学公式
mathjax: true

# Schemes #启用主题中的主题Mist
scheme: Mist

# 侧边栏
#  - post    只在文章页面显示
#  - always  所有页面显示
#  - hide    隐藏
sidebar: always

# 自动滚动到"阅读更多"标记的下面
scroll_to_more: true

# 自动给目录添加序号
toc_list_number: true

# 自动截取摘要
auto_excerpt:
  enable: false
  length: 150

# Lato 字体
use_font_lato: true

# Make duoshuo show UA
# user_id must NOT be null when admin_enable is true!
# you can visit http://dev.duoshuo.com get duoshuo user id.
duoshuo_info:
  ua_enable: true
  admin_enable: false
  user_id: 0
  #admin_nickname: ROOT

## DO NOT EDIT THE FOLLOWING SETTINGS
## UNLESS YOU KNOW WHAT YOU ARE DOING

# 动画
use_motion: true

# Fancybox 看图插件
fancybox: true

# Static files
vendors: vendors
css: css
js: js
images: images

# Theme version
version: 0.4.5.1
```
### 选择Scheme
```
scheme: Mist
```
### 添加网站图标
将网站图标文件 favicon.ico 放到source目录下面
修改主题_config.yml文件
```
favicon: /favicon.ico
```
### 菜单设置
```
menu:
  home: /
  archives: /archives
  categories: /categories
  tags: /tags
  commonweal: /404.html
  about: /about
```
### 标签页面
执行命令行
```
hexo new page "tags"
```
到生成的页面文件里
```
title: tags
date: 2015-10-4 22:37:08
type: "tags"
comments: false    #关闭评论（建议）
---
```
### 分类页面
执行命令行
```
hexo new page "categories"
```
到生成的页面文件里
```
title: categories
date: 2015-10-4 22:37:08
type: "categories"
comments: false    #关闭评论（建议）
---
```
### 关于页面
执行命令行
```
hexo new page "about"
```
到生成的页面文件里
```
title: about
date: 2015-10-4 22:37:08
type: "about"
comments: false    #关闭评论（建议）
---
```
### RSS链接
编辑主题的_config.yml文件
禁止：
```
rss: false
```
使用：
```
rss:
```
需要安装插件 hexo-generator-feed ，继续看下面有安装插件教程
自定义：
```
rss: url#自定义的地址
```
### 侧边栏设置
```
sidebar: post  #只在文章页面自动显示
#sidebar: always #在所以页面自动显示
#sidebar: hide #全隐藏
```
### 腾讯公益 404 页面
source 目录下新建 404.html 页面
```
<!DOCTYPE HTML>
<html>
<head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="robots" content="all" />
  <meta name="robots" content="index,follow"/>
</head>
<body>

<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="your-site-url" homePageName="回到我的主页"></script>

</body>
</html>
```
### 首页文章摘录
在主题配置文件添加
```
auto_excerpt:
  enable: true
  length: 150 #默认截取的长度为 150 字符
```
***

# 扩展
## 评论系统
### 多说评论系统
注册[多说](http://duoshuo.com/)，配置域名站点域名 xxx.duoshuo.com
在站点配置文件添加：
```
duoshuo_shortname: xxx
```
## 设置网站统计
### 百度统计
登陆注册[百度统计](http://tongji.baidu.com/web/welcome/login)，配置后到代码获取页面，
找到代码里 ``hm?js`` 复制后面一串字符串ID  xxx
在站点配置文件添加：
```
baidu_analytics: xxxxxxxxxxxxxxxx
```
## 分享
### 百度分享
在站点配置文件添加：
```
# 百度分享服务
baidushare: true
```
### 多说分享
在站点配置文件添加：
```
# 多说分享服务
duoshuo_share: true
```
## 版权协议
在站点配置文件添加：
```
# Creative Commons 4.0 International License.
# http://creativecommons.org/
# Available: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | zero
creative_commons: by-nc-sa
```
## 网站访问量统计
使用 [不蒜子](http://service.ibruce.info/) 提供的服务
算法a: pv的方式, 单个用户连续点击n篇文章, 记录n次访问量.
```
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
```
算法b: uv的方式, 单个用户连续点击n篇文章, 只记录1次访客数.
```
<span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>
```
找到themes/next/layout/partials/footer.wsig文件
在最后一行前面添加
```
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>
```
## 网站运行时间
在最后一行前面添加
```
<script>
var birthDay = new Date("11/20/2014");
var now = new Date();
var duration = now.getTime() - birthDay.getTime();
var total= Math.floor(duration / (1000 * 60 * 60 * 24));
document.getElementById("showDays").innerHTML = "本站已运行 "+total+" 天";
</script>
<span id="showDays"></span>
```
