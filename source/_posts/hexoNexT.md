---
title: Hexo应用NexT主题
date: 2017-04-01 20:46:49
tags:
 - Hexo
 - NexT
 - 主题
categories:
 - Hexo
---
### Hexo主题 ###


Hexo 为我们提供了很多[主题]("https://hexo.io/themes/)，本博客的主题使用的是[NexT](https://github.com/iissnan/hexo-theme-next)主题，是目前Github上Star最高的Hexo主题，支持几种不同的风格。所以本文以NexT为例，介绍Hexo主题的配置及使用。


### 安装NexT主题 ###


在 Hexo 中有两份主要的配置文件，其名称都是 _config.yml。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件。比如我的电脑下的 D:\Hexo 目录下的称为站点配置文件，D:\Hexo\themes\next 目录下的成为主题配置文件。

在终端窗口下，定位到 Hexo 站点目录下。运行下面的代码：

```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```


### 启用NexT主题 ###


与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 站点配置文件， 找到 theme 字段，并将其值更改为 next。

```
theme: next
```


到此，NexT 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 执行以下代码清除 Hexo 的缓存

```
hexo clean
```


### 验证主题 ###


首先启动 Hexo 本地站点，并开启调试模式（即加上 –debug），整个命令是 hexo s –debug。 在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：

> INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.

若运行成功，NexT 默认的 Scheme —— Muse


### 主题设定 ###


Scheme 是 NexT 提供的一种特性，借助于 Scheme，NexT 为你提供多种不同的外观。同时，几乎所有的配置都可以 在 Scheme 之间共用。目前 NexT 支持三种 Scheme

```
Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
Mist - Muse 的紧凑版本，整洁有序的单栏外观
Pisces - 双栏 Scheme，小家碧玉似的清新
```

将不需要启用的主题注释 # 掉即可。

```
# scheme: Muse
# scheme: Mist
scheme: Pisces
```


### 设置语言 ###


编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

```
language: zh-Hans
```


### 其他主题设置 ###

访问 [NexT官方设置文档](http://theme-next.iissnan.com/getting-started.html) 进行设置

如果想要在“站点概览”页面显示“友情链接”，需要在站点配置文件中加入以下代码：

```
# title, chinese available
links_title: 友情链接 
# # links
links:
   显示名称: 链接地址
```


### 集成三方工具 ###


#### 添加文章阅读量功能 ####


参考 [为NexT主题添加文章阅读量统计功能](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud) 进行设置，该博客只是针对NexT主题，其他主题请寻找其他解决办法。


#### 添加站内搜索能 ####


NexT主题支持集成 Swiftype、 微搜索、Local Search 和 Algolia,Swiftype和Algolia都只有一段时间的试用期，可以采用Hexo提供的Local Search,原理是通过hexo-generator-search插件在本地生成一个search.xml文件，搜索的时候从这个文件中根据关键字检索出相应的链接。

安装 hexo-generator-search

在站点的根目录下执行以下命令：

```
npm install hexo-generator-search --save
```


安装 hexo-generator-searchdb

在站点的根目录下执行以下命令：

```
npm install hexo-generator-searchdb --save
```


启用搜索

编辑 站点配置文件，新增以下内容到任意位置：

```
search:
	path: search.xml
  	field: post
  	format: html
  	limit: 10000
```

最后不要忘了将站点配置文件的local_search改成true。

```
# Local search
local_search:
  enable: true
```

#### 给博客添加feed ####


安装hexo-generator-feed

```
npm install hexo-generator-feed --save
```


配置到站点配置文件_config.yml

```
# Extensions
## Plugins: http://hexo.io/plugins/
#RSS订阅
plugin:
- hexo-generator-feed
#Feed Atom
feed:
type: atom
path: atom.xml
limit: 20
```

最后，在你next主题下的_config.yml下，添加RSS订阅链接即可：

```
rss: /atom.xml
```


#### 给博客生成一个站点地图 ####


安装hexo-generator-seo-friendly-sitemap

```
npm install hexo-generator-seo-friendly-sitemap --save
```


在站点配置文件_config.yml 中添加

```
sitemap:
	path: sitemap.xml
```

#### 头像圆形旋转 ####


把完整的 [sidebar-author.styl](https://github.com/ehlxr/useful-code/blob/master/resources/sidebar-author.styl) 文件内容复制替换到

```
..hexo/themes/next/source/css/_common/components/sidebar/sidebar-author.styl
```



Read More:

> [Hexo博客添加站内搜索](http://www.ezlippi.com/blog/2017/02/hexo-search.html) 
> [记录Hexo+Github免费搭建个人博客](https://superbsco.coding.me/2017/01/13/new-article/)