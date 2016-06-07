---
layout: post
title: 'Django框架学习中的问题（二）'
date: 2014-11-13 15:19
comments: true
categories: 
---
#Django的工作原理
感觉大多数框架形形色色的思路基本都是一致的，就是哪些框架更灵活，哪些框架的去侵入性做的更好，哪些框架的中间件的拔插方式更方便，哪些框架的语法糖更易用，哪些框架带来了更大的生产力。
像Java中的SSH，基本都是带侵入式的，比如你要把Hibernate换成Mybatis基本上会头疼致死，而Django只需要修改下settings.py文件。
像Java中你需要添加很多拦截器来起到中间件效果，通过拦截器也使这些拦截器不再耦合，不过这些中间件基本上要针对你MVC各层框架，及持久化层框架，及业务来编写。而Django中这些基本的中间件基本上都有成熟的库，只需要针对业务做一些自己实现的中间件，这点很类似于node.js的express框架。
像Java中的路由控制，你需要通过struts或者SpringMVC，都需要编写Action（或者controller），经常会把业务集成在Action中，Django就更简单了点，urls.py只负责正则映射，调用指定的方法去做处理。职责更分明，做的事情也自然单一了。
而数据库的访问Django已经封装好了，只需要调用Model的相应方法即可，而业务逻辑也应当封装在Model中，Views中的函数只负责完成程序逻辑。
而视图层其实在各种框架中都差不多，基本都是模板加程序语言去实现。比如Django中有Django的模板方法，而ejs、jade、velocity、freemaker、jsp都有相应的方法。
Django的大致工作原理如下图：
![](http://chuantu.biz/t/17/1415892562x-1376440137.jpg)

#几个命令
创建Django工程：django-admin.py startproject projectname
工程中创建新的APP：mange.py startapp appname
运行工程：mange.py runserver (None port or 0.0.0.0:prot)
在工程中同步数据库：mange.py syncdb

#python对应的数据库模块未安装
```python settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'TestDjango',
        'PASSWORD': '123',
        'HOST':'',
    }
}
```
运行命令后报错为： windows Error loading MySQLdb module: No module named MySQLdb
原因：python2.7不会自动安装对应的MySQLdb
[下载地址](http://www.codegood.com/download/11/)
安装到python的site-package目录下就OK了。

#用户漏了配置
配置后Access denied for user 'ODBC'@'localhost' (using password: YES)")
原因：漏了配置用户名
改成如下代码就能正常了。
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'root',
        'NAME': 'TestDjango',
        'PASSWORD': '123',
        'HOST':'',
    }
}
```
#POST提交表单，action需要/结尾
报错：You called this URL via POST, but the URL doesn't end in a slash and you have APPEND_SLASH set
原因：action属性没有结尾，如下，/login/，最后的斜杠不能漏

```html
<form class="navbar-form navbar-right" action="/login/" method="post" role="form">
<div class="form-group">
<input type="text" name="username" placeholder="用户名" class="form-control">
</div>
<div class="form-group">
<input type="password" name="password" placeholder="密码" class="form-control">
</div>
<button type="submit" class="btn btn-success">登 录</button>
</form>
```
#还是提交表单，又报错了
报错：CSRF verification failed. Request aborted.
原因：这是一个跨单访问的错误，去掉相应的中间件即可,看下列代码注释
```
MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    #'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.security.SecurityMiddleware',
)
```
如果需要使用跨单访问保护，又不想报错，那就需要在表单中加\{\% csrf_token \%\}，我没试过，先记下来需要时方便查询。
