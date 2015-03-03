title: Python_Django_配置安装
date: 2015-1-31 18:34:22
categories: Python
---
Mac下Djaongo开发环境配置

<!--more-->
安装django框架

	sudo easy_install django


测试下安装成功没有
	
	python
	>>> import django
	>>> print (django.get_version())
	1.7

创建一个工程

	django-admin.py startproject helloDjango
	django-admin startapp blog
	
配置app
	
	vim ./helloDjango/settings.py
	 add-->blog
	
配置url
	
	vim urls.py
	url(r'blog/index/$','blog.views.index')
	
写hello wrold
	
	vim blog/views.py
	from django.http import HttpResponse
	def index(request):
		return HttpResponse('hello Django!')
	
跑起来
	
	python manage.py runserver
	#浏览器访问 127.0.0.1:8000 ，看到hello Django！大功告成。
	