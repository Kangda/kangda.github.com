---
date: 2013-03-23 17:49:43+00:00
layout: post
title: Linux下Latex（Xelatex）环境配置
categories: blog
tags:
- latex
- linux
- xelatex
- xetex
description: Ubuntu环境下XeLatex的安装配置
---

### 软件环境：





	
  * Ubuntu 12.04

	
  * XeTeX 3.1415926-2.2-0.9995.2 (TeX Live 2009/Debian)

	
  * Texmaker 3.2




### 安装：


应该是可以直接下载Texlive的iso然后安装，不过那个可能大一些，现在最新的貌似是Texlive2012

在Ubuntu上，可以从源里安装，当然源里的旧一些，就像上面的环境写得一样，貌似是Texlive2009

主要要装的Texlive、Texlive的一些扩展、XeLatex、Texmaker。Texlive一般包含了XeTex，而XeLatex是XeTex中Latex的部分，而XeTex主要特点是支持了Unicode。Texmaker是一个Latex的编辑器，跨平台。

	sudo apt-get install texlive texlive-latex-extra texmaker


### 配置：


要在Texmaker中配置XeLatex，在Option>> Configure Texmaker>> Quick Build>> User中前面添上

xelatex -interaction=nonstopmode %.tex|

ps: 最后有一个小竖线，为了和其他已有的内容区分开


### 测试：


最初想到用Latex就是因为要造一个简历，所以这里拿简历的样本来说了

简历模板用的是moderncv，贴一个文件头上的设置部分吧


	\documentclass[10pt,a4paper]{moderncv}
	% moderncv themes
	\moderncvtheme[blue]{classic} % optional argument are 'blue' (default), 'orange', 'red', 'green', 'grey' and 'roman' (for roman fonts, instead of sans serif fonts)
	%\moderncvtheme[green]{classic} % idem
	% character encoding
	\usepackage{xeCJK} % replace by the encoding you are using
	\setmainfont[Mapping=tex-text]{WenQuanYi Micro Hei}
	\setsansfont[Mapping=tex-text]{WenQuanYi Micro Hei}
	\XeTeXlinebreaklocale "zh"
	\setCJKmainfont{WenQuanYi Micro Hei}

