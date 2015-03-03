title: 在github上搭建hexo博客
date: 2014-10-04 20:47:57
categories: hexo
---
折腾了一下午终于把hexo框架弄好了，刚换mac有点不习惯，node.js的环境配置捣鼓了半天。其实还是蛮简单的，熟练的话几分钟就可以配置到github上面去了。
 <!--more-->


#步骤
###安装Git
这很简单。下载直接运行，傻瓜式。[Down Git for MAC](https://mac.github.com)

###安装node.js
下载pkg安装，安装默认路径就可以了。[Down nodejs for MAC](http://www.nodejs.org/download/)

###配置nodejs环境变量

在终端执行命令node -v查看nodejs环境变量正确与否，如果不正确，在终端输入

```
echo "export PATH=/bin:/sbin/:/usr/bin:/usr/sbin:/usr/local/bin:/opt/local/bin:/opt/local/sbin:$PATH" >> ~/.bash_profile
```

###安装hexo

```
npm install -g hexo
```

###创建hexo文件夹

```
hexo init blog
```

###安装依赖包

```
npm install
```

###部署至github

```
hexo generate
hexo server
```

编辑_config.yml。

```
deploy:
  type: github
  repository: https://github.com/ihgoo/ihgoo.github.io.git
  branch: master
```

  执行hexo g，hexo d，完成部署。在github上成功上传hexo。
