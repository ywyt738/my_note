*****************
第一课
*****************

* Django版本:1.11.2
* Pthon版本:3.6.1

安装
=============

::

  pip install django

开始
=============
创建项目::

  django-admin startproject mysite

项目结构::

  G:\MYSITE
  │  manage.py
  │
  └─mysite
     │  __init__.py
     │  urls.py
     │  wsgi.py
     └─ settings.py

开发服务器(在 manage.py 文件目录下运行)::

  python manage.py runserver
  python manage.py runserver 8080    #8080端口运行
  python manage.py runserver 0:8000  #监听所有网卡ip

创建app::

  python manage.py startapp polls

app结构::

  polls
  │  __init__.py
  │  admin.py
  │  apps.py
  │  models.py
  │  tests.py
  │  urls.py
  ├─ views.py
  │
  └─ migrations
         __init__.py

写第一个view
=====================

polls/views.py

.. code-block:: python

  from django.http import HttpResponse

  def index(request):
      return HttpResponse("Hello, world. You're at the polls index.")

写url指向这个view

polls/urls.py

.. code-block:: python

  from django.conf.urls import url
  from . import views

  urlpatterns = [
      url(r'^$', views.index, name='index'),
  ]

在root URLconf里调用polls.url模块

mysite/urls.py

.. code-block:: python

  from django.conf.urls import include, url
  from django.contrib import admin

  urlpatterns = [
      url(r'^polls/', include('polls.urls')),
      url(r'^admin/', admin.site.urls),
  ]

**注意:** include方法的正则表达式不能以 ``$`` 结尾。
