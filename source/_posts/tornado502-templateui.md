---
title: Tornado5.0.2翻译文档 - 模板和UI
date: 2020-12-19 12:34:43
tags:
 - Tornado
 - 翻译文档
categories:
 - Tornado
---

Tornado包含一种简单、快速、灵活的模板语言。本节描述该语言以及相关问题，如国际化。

Tornado还可以与任何其他Python模板语言一起使用，尽管没有将这些系统集成到`RequestHandler.render`中。只需将模板呈现为一个字符串并将其传递给`RequestHandler.write`即可。

### 模板配置

默认情况下，Tornado会在引用它们的`.py`文件的同一目录中寻找模板文件。要将模板文件放到不同的目录中，请使用`template_path`应用程序设置(或覆盖`RequestHandler.get_template_path`如果不同的处理程序有不同的模板路径)。

要从非文件系统位置加载模板，子类`tornado.template.BaseLoader`并传递一个实例作为`template_loader`应用程序设置。

编译后的模板默认缓存;要关闭此缓存并重新加载模板，以便底层文件的更改始终可见，请使用应用程序设置`compiled_template_cache=False`或`debug=True`。

### 模板语法

Tornado模板只是HTML(或任何其他基于文本的格式)，在标记中嵌入了Python控制序列和表达式:

```
<html>
   <head>
      <title>{{ title }}</title>
   </head>
   <body>
     <ul>
       {% for item in items %}
         <li>{{ escape(item) }}</li>
       {% end %}
     </ul>
   </body>
 </html>
```

如果您将此模板保存为“template.html”。然后把它放在与Python文件相同的目录下，你可以用下面的方法来渲染这个模板:

```
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        items = ["Item 1", "Item 2", "Item 3"]
        self.render("template.html", title="My title", items=items)
```






Read More:

> [Templates and UI](https://www.tornadoweb.org/en/stable/guide/templates.html)





