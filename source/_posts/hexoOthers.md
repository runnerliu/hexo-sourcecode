---
title: Heox主题的其他配置
date: 2017-04-01 21:34:40
tags:
 - Hexo
 - 主题
 - 博客
categories:
 - Hexo
---

### Hexo页面底部次数显示

很多网站中都有访问人数和总访问量，也就是下图所示的功能：

![截图20170402095045](/images/2017-4-2 102013.png)

我们可以通过在主题配置文件中添加以下代码来实现此功能：

```
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"></i> 访问人数
  site_uv_footer:
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"></i> 总访问量
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"></i> 浏览
  page_pv_footer: 次
```

### 添加社交链接

如果希望试下下图显示的功能：

![截图20170402095414](/images/2017-4-2 102023.png)

可以在主题配置文件中添加如下代码：

```
social:
  #LinkLabel: Link
  GitHub: 链接地址
  weibo: 链接地址
  zhihu: 链接地址


# Social Links Icons
# Icon Mapping:
#   Map a menu item to a specific FontAwesome icon name.
#   Key is the name of the item and value is the name of FontAwesome icon. Key is case-senstive.
#   When an globe mask icon presenting up means that the item has no mapping icon.
social_icons:
  enable: true
  # Icon Mappings.
  # KeyMapsToSocialItemKey: NameOfTheIconFromFontAwesome
  GitHub: github
  weibo: weibo
  zhihu: hand-o-right
```

Hexo主题使用的图标都在 [FontAwesome](http://fontawesome.io/) 上，你可以根据自己需要选择，我发现这是一个很不错的网站，各种图标很漂亮，PPT中也可以用。

### 分类、归档、关于页面的创建

安装完成Hexo之后，我们点击“分类”、“关于”、“标签”页面都会显示GitH的404页面，这是因为我们还没有创建每个功能对应的文件，使用如下方法创建：

分类：

```
hexo new page "categories"
```

关于：

```
hexo new page "about"
```

标签：

```
hexo new page "tags"
```

使用以上hexo命令会在Hexo根目录的source文件夹下创建各个对应的文件夹，每个文件夹中都有index.md文件，这就成功了，文件中的内容不用修改。

创建完成之后，我们在编辑文章时：

```
title: 标题
date: 2017-04-01 21:34:40
tags:
 - Hexo
 - 主题
 - 博客
categories:
 - Hexo
```

“tags”、“categories”是对文章打的标签和分类，默认文章是可以评论的，如果不希望评论，可以加上：

```
comments： false
```
### 首页文章显示摘要

第一种方式：在主题配置文件中，找到auto_excerpt，修改如下：

```
# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: true
  length: 150
```

第二种方式：根据文章的内容，自己在合适的位置添加`<!--more-->`标签，使用灵活，也是Hexo推荐的方法

![img](http://upload-images.jianshu.io/upload_images/2352140-ec93f0ac69d07b21.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三种方式：在文章中的`front-matter`中添加description，并提供文章摘录。这种方式只会在首页列表中显示文章的摘要内容，进入文章详情后不会再显示。

![img](http://upload-images.jianshu.io/upload_images/2352140-67c6e1edb5695035.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)