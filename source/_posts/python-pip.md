title: Python-pip源
date: 2015-04-22 14:47:58
categories: Python
---
使用pip时会遇到Cannot fetch index base URL https://pypi.python.org/simple/，可以设置其他镜像源。

<!--more-->

###pipy国内镜像目前有：
---
- http://pypi.douban.com/  豆瓣

- http://pypi.v2ex.com/simple V2EX

- http://pypi.hustunique.com/  华中理工大学

- http://pypi.sdutlinux.org/  山东理工大学

- http://pypi.mirrors.ustc.edu.cn/  中国科学技术大学


###手动指定源
---
在pip后面跟-i 来指定源，比如用豆瓣的源来安装web.py框架：

pip install djangorestframework -i http://pypi.v2ex.com/simple

###修改配置文件
---
需要创建或修改配置文件（linux的文件在~/.pip/pip.conf，windows在%HOMEPATH%\pip\pip.ini），修改内容为：

	[global]
	index-url = http://pypi.douban.com/simple