---
date: 2012-08-13 10:11:47+00:00
layout: post
title: Sublime Text 2
categories: blog
description: Sublime Text2安装与配置
---

发现了Sublime Text2这样的一个编辑器，它比Vim更容易学习，但是也很华丽，同时还能支持扩展，应该说是我这种比较懒，学Vim很没兴趣的人的不错选择。



[Sublime Text 2](http://www.iplaysoft.com/sublimetext.html) 除了自身拥有无数实用功能和特性之外，它还能安装使用各种扩展/皮肤/配色方案等来增强自己。

安装Package Control：

按键盘 Ctrl+`（数字1左边的按键）调出控制台，然后拷贝下面的代码进去并回车

    
    import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'




这里主要记一下ST2的一些快捷键：

全局热键：



	
  * Ctrl+Shift+P 命令输入

	
  * Ctrl+鼠标左键 多项选择，并且可以实现多处同时编辑

	
  * Ctrl+D 将全文中与光标当前所在位置的词相同的词逐一加入选

	
  * Ctrl+F 搜索

	
  * Ctrl+S 保存

	
  * Ctrl+R 搜索函数

	
  * Ctrl+G 跳转到特定行




Terminal





	
  * Ctrl+Shift+T 开启一个当前文件路径下的终端

	
  * Ctrl+Shift+Alt+T 开启当前功能工程下的终端


Alignment

	
  * Ctrl+Alt+A 使所选部分以等号对齐


Python PEP8 Autoformat

	
  * Ctrl+Shift+R 格式化当前代码


