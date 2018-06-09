---
title: Hexo原理及其常用命令
date: 2017-04-02 10:35:08
tags:
 - Hexo
 - 原理
 - 命令
categories:
 - Hexo
---

### Hexo原理

2017年4月1日使用Hexo+GitHub创建了自己的博客，主要用于生活和学习总结，经过两天的折腾，基本的博客雏形已经出来了，不得不说GitHub Pages真是个好东西。给赞。

对于Hexo，[官网文档](https://hexo.io/docs/) 解释为：

> Hexo is a fast, simple and powerful blog framework. You write posts in [Markdown](http://daringfireball.net/projects/markdown/) (or other languages) and Hexo generates static files with a beautiful theme in seconds.

**generates static files** 这就是Hexo的作用所在，我们都知道GitHub给了广大程序设计人员一个存放开源项目的平台，每个人都可以迭代更新自己的项目或参与开发其他人的项目，而GitHub Pages就是给你提供了一个平台来显示你博客的静态页面，注意是**静态页面**，因为GitHub Pages不支持动态语言，只能使用 html 拼合成博客。

Hexo就是这个生成静态页面的框架，流程如下

```
markdown**.md -> hexo -> **.html -> github -> update website
```

你在本地编写并用Hexo生成了静态页面，存储在/public 目录下，然后push到GitHub上，GitHub展示了你的页面而已。

根据[Hexo的原理是什么](https://www.zhihu.com/question/51588481) 问题中[孔晨皓](https://www.zhihu.com/people/kong-chen-hao) 的回答：

> 如果要做博客 wordpress 的思路是 php + MySql 而 gitpages 不支持动态语言，因此只能使用 html 拼合成博客
>
> 首先自己本地文件夹的 source 就是数据库，以 .md(markdown) 格式存储文章，
>
> theme 文件夹是主题文件，以 .yml 等类型，决定了页面如何“组装”
>
> 每次运行 hexo g 命令，hexo(node.js程序)会遍历你的 source 目录，建立索引，根据你 theme 文件夹的主题生成页面到 public 文件夹。这时 public 文件夹就是一个纯由 html javascript css 等内容制作的博客，而这些恰好能在 git pages 识别

而我们直接打开/public 目录中的**.html 是不能直接显示的，需要在命令行中执行以下命令：

```
hexo s
```

这条命令相当于hexo通过nide.js启动了一个本地服务器。

### Hexo常用命令

详见[hexo常用命令笔记](https://segmentfault.com/a/1190000002632530) 