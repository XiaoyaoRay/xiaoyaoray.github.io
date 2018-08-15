---
title: 解决Centos7开启Django服务不能远程访问的问题
copyright: true
date: 2018-06-01 09:55:21
categories:
- Python
tags:
- Python
- Django
---

**转载**：https://blog.csdn.net/dazhi_1314/article/details/78325506

### 修改启动命令

```
python manage.py runserver 0.0.0.0:<port>
```

外网和127.0.0.1都能够访问
<!--more-->
### 问题解决

可能会出现DisallowedHost at / Invalid HTTP_HOST header: 
DisallowedHost at / 
Invalid HTTP_HOST header: ‘x:8000’. You may need to add u’10.211.55.6’ to ALLOWED_HOSTS.

Request Method: GET 
Request URL: <http://x:8000/> 
Django Version: 1.10.4 
Exception Type: DisallowedHost 
Exception Value: 
Invalid HTTP_HOST header: ‘10.211.55.6:8000’. You may need to add u’10.211.55.6’ to ALLOWED_HOSTS. 
Exception Location: /usr/lib/python2.7/site-packages/django/http/request.py in get_host, line 113 
Python Executable: /usr/bin/python 
Python Version:

如果出现上述问题，可以进入django-admin.py startproject project-name创建的项目中去修改 settings.py 文件： 
ALLOWED_HOSTS = [‘*’] ＃在这里请求的host添加了＊ ，再运行下面的命令，就可以远程访问django服务器了

```
python manage.py runserver 0.0.0.0:<port>
```
