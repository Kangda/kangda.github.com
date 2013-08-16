---
date: 2012-06-01 15:12:01+00:00
layout: post
title: Django Things
categories: blog
tags:
- django
- python
description: Django扎记
---

Q: 如何对静态文件（如.js,.css文件）进行设置？

A: 在开发时，即使用django的runserver时，可以做如下设置：

1.	在settings出设置STATICFILES_DIRS，把每个app中的static的路径都添加进去，要求是绝对路径;

2.	在url中设置如下：

		if settings.DEBUG:
			urlpatterns = static(settings.STATIC_URL)

3.	重新运行runserver检验
	在产品运行时，要对webserver进行设置，暂时还未研究



Q:form对象与其内定义的field是什么关系，如果访问？

A: 经过实践证明，form对象中定义的field和model中定义的那些field有不同，form中的field不能直接通过对象后面的‘.’来访问field，而要通过form的成员变量fields字典来访问，如：form中有一个field名叫f1,则要通过form对象访问它的时候就要通过form.fields['f1']来获取，而**不能直接通过**form.f1来访问



Q:对于自定义的Choices的ChoiceField，在校验时显示invalid choice

A:其他表单进行校验时，往往是从request.POST中生成form对象，但是由于这里是自定义的Choices，所以每次Choices都要动态生成，不能从request.POST中生成，所以在校验的时候由于Choices为空，所以校验都会显示invalid choice，要使得校验通过，则需要在request.POST生成form对象后再手动定义Choices


