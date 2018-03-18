---
title: Sublime Text 3 添加右键菜单
date: 2017-10-11 19:38:35
tags:
 - Sublime Text 3
categories:
 - 辅助研发杂记
---

每次安装 Sublime Text 3 后邮件菜单总是没有，但是 notepad++ 安装后会自动加到邮件菜单，对于 Sublime Text 3 的忠实粉丝来说简直丧心病狂，所以搜罗了一些方法来完成这件事儿。

#### 方式1

```
[Version]
Signature="$Windows NT$"

[DefaultInstall]
AddReg=SublimeText3

[SublimeText3]
hkcr,"*\\shell\\SublimeText3",,,"Edit with Sublime Text 3"
hkcr,"*\\shell\\SublimeText3\\command",,,"""%1%\sublime_text.exe"" ""%%1"" %%*"
hkcr,"Directory\shell\SublimeText3",,,"Edit with Sublime Text 3"
hkcr,"*\\shell\\SublimeText3","Icon",0x20000,"%1%\sublime_text.exe, 0"
hkcr,"Directory\shell\SublimeText3\command",,,"""%1%\sublime_text.exe"" ""%%1"""
```

把以上代码，复制到 Sublime Text 3 的安装目录，然后重命名为：sublime_addright.inf，注意文件编码方式为ANSI，右击安装就可以了。

> 重命名文件之前，需要先在 工具->文件夹选项->查看，把隐藏已知文件类型的扩展名前边的复选框不勾选。

#### 方式2

```
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\*\shell\SublimeText3]
@="Edit with Sublime Text 3"
"Icon"="D:\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\*\shell\SublimeText3\command]
@="D:\\Sublime Text 3\\sublime_text.exe %1"


[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
@="Edit with Sublime Text 3"
"Icon"="D:\\Sublime Text 3\\sublime_text.exe,0"

[HKEY_CLASSES_ROOT\Directory\shell\SublimeText3\command]
@="D:\\Sublime Text 3\\sublime_text.exe %1"
```

把以上代码，复制到 Sublime Text 3 的安装目录，然后重命名为：sublime_addright.reg，注意文件编码方式为ANSI，然后双击就可以了。

> 需要把里边的 Sublime 的安装目录，替换成实际的 Sublime 安装目录。

删除右键菜单脚本：

```
Windows Registry Editor Version 5.00
[-HKEY_CLASSES_ROOT\*\shell\SublimeText3]
[-HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
```

把以上代码，复制到 Sublime Text 3 的安装目录，然后重命名为：sublime_delright.reg，文件编码方式依然为ANSI，然后双击就可以了。



Read More：

> [将Sublime Text3添加到右键菜单中](https://my.oschina.net/adairs/blog/466777)