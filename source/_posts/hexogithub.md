---
title: Hexo+Github搭建个人博客
date: 2017-04-01 17:31:17
tags:
 - Hexo
 - GitHub
 - 博客
categories:
 - Hexo
---

### 动机 ###

促使我建立自己博客的动机很简单，其他的博客平台太丑了！简直不能看啊，然而我并没有在说CSDN、CNBLOG等等。还有一个原因就是，我有一颗向往技术大神的心，所以需要靠自己一点点的积累。在过滤各种博客平台的时候，看到某乎大神建议使用GitHub搭建自己的博客，So，心动不如行动，于是搜集各种资料，搭建了自己博客，本文主要是记录搭建的过程以及可能遇到的问题及解决办法。  
> **本教程只适用于windows操作系统**

### 准备工具 ###


- [GitHub](https://github.com/)账户配置  
- 安装[Node.js](https://nodejs.org/en/)环境  
- 安装[Git](https://git-scm.com/)环境


#### GitHub账户配置 ####

[GitHub](https://github.com/)账户申请：  


- 如果已经拥有GitHub账户，请进行第二步注册结束后，一定前往自己的注册邮箱，点开GitHub发送给你的注册确认信，确认注册，结束注册流程。否则无法使用gh-pages。


创建代码库：


- 登陆之后，点击页面右上角的加号，选择New repository新建代码库；  
  进入代码库创建页面，在Repository name下填写yourname.github.io，Description (可选)下填写一些简单的描述（不写也可以）；
- **注意**：比如我的github名称是runnerliu,这里就填 [runnerliu.github.io](runnerliu.github.io)；


代码库设置:


- 开启gh-pages功能，正确创建代码库后，点击界面右侧的Settings，你将会打开这个库的setting页面，向下拖动，直到看见GitHub Pages；
- 点击Automatic page generator，Github将会自动替你创建出一个gh-pages的页面。 如果配置没有问题，那么大约15分钟之后，yourname.github.io这个网址就可以正常访问了；**注意**：我自己在操作过程中并没有这一步，可能是GitHub升级了？


#### 安装Node.js环境 ####


根据自己操作系统的版本前往[Node.js](https://nodejs.org/en/)下载相应的Node.js，可以根据所需选择安装路径，默认安装的话一路next就可以。  


验证安装：


- 打开控制台（Win+R输入cmd，回车）；
- 输入`node -v`查看是否输出版本号；
- 输入`npm -v`查看是否输出版本号；
- 如果全部输出成功，证明安装成功，否则检查系统环境变量是否配置。


#### 安装Git环境 ####


前往[Git](https://git-scm.com/)官网下载相应版本的Git；  

和Node.js一样，根据需要选择安装路径，可以一路默认安装；  

安装完成后，在控制台中输入`git --version`查看是否添加了系统环境变量，这样就可以直接在dos中使用git命令了；  如果输出有误，检查git环境变量的设置。    


> 完成以上步骤，准备阶段的工作就完成了。


### Hexo ###


- 安装Hexo
- 配置Hexo
- 将Hexo与github page绑定
- Hexo的使用设置


#### 安装Hexo ####


在合适的地方创建一个文件夹，这里我以D：/hexo 为例讲解，首先在D盘目录下创建Hexo文件夹，并在命令行的窗口进入到该目录；

在命令行中输入`npm install hexo-cli -g`，如果看到WARN也别担心，不影响使用；

继续输入`npm install hexo --save`；

稍等安装完成后，输入`hexo -v`查看Hexo是否安装成功。如果安装成功会显示出hexo的版本号以及其他信息；


#### 配置Hexo ####


初始化Hexo：


- 在DOS中输入`hexo init`；
- 然后输入`npm install`，npm将会自动安装你需要的组件，只需要等待npm操作即可。


体验Hexo：


- 在DOS中输入`hexo g`，等待完成；
- 然后输入`hexo s`，如果提示`INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`，证明Hexo启动成功；
- 在浏览器中打开http://localhost:4000/，会看到Hexo的效果。  


#### 将Hexo与github page绑定 ####


配置Git个人信息


- 设置Git的user name和email：(如果是第一次的话)
- 运行`git config --global user.name "zhangsan"`，"zhangsan"是你GitHub账户名；
- 运行`git config --global user.email "zhangsan@163.com"`，"zhangsan@163.com"是你注册GitHub的邮箱。


使用SSH远程连接GitHub库（本部分命令都在Git Bash中执行）


- **注意**：配置SSH主要是用于本地远程更新GitHub代码库，在搜集资料的时候看到有人说可以用https url的方式，但是我实践过程中是不行的，而且https url的方式在fetch和push代码都需要输入账号和密码，这是比较麻烦的。建议使用SSH-Key方式。
- 打开Git Bash，输入`ssh-keygen -t rsa -C "zhangsan@163.com" -f "filename"`生成密钥，"zhangsan@163.com"是你注册GitHub的邮箱，然后**连续三个回车**；
- 最后得到了两个文件：id\_rsa和id\_rsa.pub，默认存数在`C:\Users\电脑名称\.ssh`；
- 添加密钥到ssh-agent：输入 `eval "$(ssh-agent -s)"` ，输入 `ssh-add ~/.ssh/id_rsa` ；
- 登录GitHub，点击自己头像选择"Settings"，选择"SSH and GPG keys"，选择"New SSH key"，将id_rsa.pub中的内容copy过去；
- 在Git Bash终端中输入 `ssh -T git@github.com` 验证SSH-key是否配置成功，如果出现 `Hi zhagnsan! You’ve successfully authenticated, but GitHub does not provide shell access.` 表明配置成功；


配置Deployment


- 在Hexo根目录下的_config.yml文件中，找到Deployment，然后按照如下修改：

  ```
  deploy:
  	type: git
  	repo: git@github.com:yourname/yourname.github.io.git
  	branch: master
  ```


#### Hexo的使用设置 ####


首先安装扩展： `npm install hexo-deployer-git --save` ；

新建一篇博客，执行下面的命令： `hexo new post "article title"` ，这时候在电脑的目录下 D:\Hexo\source\_posts 将会看到 article title.md 文件；

用MarDown编辑器打开就可以编辑文章了。文章编辑好之后，运行生成、部署命令：

```
hexo g   // 生成
hexo d   // 部署 

或者 hexo d -g //在部署前先生成
```

然后访问 https://yourName.github.io/ 查看生成的博客。  


#### 站点配置信息 ####

在Hexo根目录下的\_config.yml文件中，根据需要配置各项信息:

```
博客名称
title: 我的博客
副标题
subtitle: 一天进步一点
简介
description: 记录生活点滴
博客作者
author: John Doe
博客语言
language: zh-CN
时区
timezone:
博客地址,与申请的GitHub一致
url: http://zhangsan.github.io
root: /
博客链接格式
permalink: :year/:month/:day/:title/
permalink_defaults:
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
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
default_category: uncategorized
category_map:
tag_map:
日期格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss
分页，每页文章数量
per_page: 10
pagination_dir: page
博客主题
theme: 
发布设置
deploy: 
  type: git
  repository: https://github.com/zhangsan/zhangsan.github.io.git
  branch: master
```



Read More:

> [手把手教你用Hexo+Github 搭建属于自己的博客](http://blog.csdn.net/gdutxiaoxu/article/details/53576018) 
> [Git ssh 配置及使用](http://blog.csdn.net/gdutxiaoxu/article/details/53573399)