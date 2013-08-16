---
date: 2012-07-28 17:25:38+00:00
layout: post
title: jquery加载页面的方法
categories: blog
tags:
- jquery
- web
description: 用jquery实现下拉自动加载页面
---

1、

	$(function(){
	//adding your code here
	});

2、

	$(document).ready(function(){
	//adding your code here
	});

3、

	window.onload = function(){
		//adding your code here
	});

页面需要引用jquery的js文件

一般的加载页面时调用js方法如下：

	window.onload = function() {
		$("table tr:nth-child(even)").addClass("even"); //这个是jquery代码
	};

这段代码会在整个页面的document全部加载完成以后执行。不幸的这种方式不仅要求页面的DOM tree全部加载完成，而且要求所有的外部图片和资源全部加载完成。更不幸的是，如果外部资源，例如图片需要很长时间来加载，那么这个js效果就会让用户感觉失效了。

但是用jquery的方法：

	$(document).ready(function() {
	// 任何需要执行的js特效
		$("table tr:nth-child(even)").addClass("even");
	});

就仅仅只需要加载所有的DOM结构，在浏览器把所有的HTML放入DOM tree之前就执行js效果。包括在加载外部图片和资源之前。

还有一种简写的方式：

	$(function() {
	// 任何需要执行的js特效
		$("table tr:nth-child(even)").addClass("even");
	});
