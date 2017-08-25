*****************
第二课
*****************

设置数据库
=================

1. 在mysite/setting.py中的Databases设置相关参数

支持的数据库

* PostgreSQL，需要 `psycopg2 <http://initd.org/psycopg/>`_，详细参考 `PostgreSQL notes <https://docs.djangoproject.com/en/1.11/ref/databases/#postgresql-notes>`_
* MySQL，需要 `DB API driver <https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-db-api-drivers>`_ 例如：**mysqlclient**。详情参考 `MySQL backend注意事项 <https://docs.djangoproject.com/en/1.11/ref/databases/#mysql-notes>`_
* SQLite，参考 `SQLite backend注意事项 <https://docs.djangoproject.com/en/1.11/ref/databases/#sqlite-notes>`_
* Oracle，需要 `cx_Oracle <https://oracle.github.io/python-cx_Oracle/>`_，详细参考 `Oracle backend注意事项 <https://docs.djangoproject.com/en/1.11/ref/databases/#oracle-notes>`_

修改 **DATABASE 'default'** 可以改变数据库的类型。

* **ENGINE**-支持 **'django.db.backends.sqlite3'** , **'django.db.backends.postgresql'** , **'django.db.backends.mysql'** ,或者 **'django.db.backends.oracle'**
* **NAME**-数据库名字。

还需要设置：**USER**, **PASSWORD**, **HOST**

2. 确保自己的app在 **INSTALLED_APPS**

初始化数据库表::

  $ python manage.py migrate

创建Models
====================

在polls应用中创建2个models， **Question** 和 **Choice** 。一个 **Question** 有一个question和一个发布日期。一个 **Choice** 有两个字段，选择项文本和一个投票计数，每个 **Choice** 可以关联一个 **Question** 。

polls/models.py

.. code-block:: python

  from django.db import models

  class Question(models.Model):
      question_text = models.CharField(max_length=200)
      pub_date = models.DateTimeField('date published')


  class Choice(models.Model):
      question = models.ForeignKey(Question, on_delete=models.CASCADE)
      choice_text = models.CharField(max_length=200)
      votes = models.IntegerField(default=0)

激活Models
=====================

需要将apps加入到project中来，需要在setting文件中的 **INSTALLED_APPS** 里指明。这里PollsConfig类是在 **polls/apps.py** 里，所以路劲是 **polls.apps.PollsConfig** 。

mysite/settings.py

.. code-block:: python

  INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    ]

现在Django知道包含了 **polls** 应用，运行如下命令::

  $ python manage.py makemigrations polls

通过 **makemigrations** 告诉Django，你已经修改了models。

运行下面的命令能看到models修改的sql过程::

  $ python manage.py sqlmigrate polls 0001

Playing with the API
==========================
使用以下命令进入Django命令行交互模式::

  $ python manage.py shell

使用这个命令代替"python"的原因是 **manage.py** 能设置 **DJANGO_SETTING_MODULE** 环境变量。

开始探索 `database API <https://docs.djangoproject.com/en/1.11/topics/db/queries/>`_


.. code-block:: python

  # 导入我们刚刚创建的model。
  >>> from polls.models import Question, Choice

  # 现在还没有Question在我们的数据库里。
  >>> Question.objects.all()
  <QuerySet []>

  # 创建一个新的Question。
  # 默认settings文件里已经支持了时区，
  # 因此Django希望pub_date是带有时区的时间格式。
  # 用timezone.now()代替datetime.datetime.now()
  >>> from django.utils import timezone
  >>> q = Question(question_text="What's new?", pub_date=timezone.now())

  # 保存这个object进入数据库，你只需使用save()
  >>> q.save()

  # 现在它有了个ID。注意这里有可能用"1L"来代替"1"，这取决于你的正在使用的数据库。
  >>> q.id
  1

  # 通过Python的属性来访问model的字段
  >>> q.question_text
  "What's new?"
  >>> q.pub_date
  datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

  # 通过改变属性来更改值，然后使用save()
  >>> q.question_text = "What's up?"
  >>> q.save()

  # object.all()会显示所有数据库里的数据。
  >>> Question.objects.all()
  <QuerySet [<Question: Question object>]>

**<Question: Question object>** 看起来不怎么方便。修改 **Question** model（在 **polls/models.py** ）并且增加
一个 **__str__** 方法就显示的更加友好。

polls/models.py:

.. code-block:: python

  from django.db import models
  from django.utils.encoding import python_2_unicode_compatible

  @python_2_unicode_compatible  # only if you need to support Python 2
  class Question(models.Model):
      # ...
      def __str__(self):
          return self.question_text

  @python_2_unicode_compatible  # only if you need to support Python 2
  class Choice(models.Model):
      # ...
      def __str__(self):
          return self.choice_text

这是一个常规的Python方法，我们来自建一个方法

polls/models.py:

.. code-block:: python

  import datetime

  from django.db import models
  from django.utils import timezone


  class Question(models.Model):
      # ...
      def was_published_recently(self):
          return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

保存，通过 **python manage.py shell** 重新打开一个Django命令行:

.. code-block:: python

  >>> from polls.models import Question, Choice

  # 确认一下__str__()是否工作正常。
  >>> Question.objects.all()
  <QuerySet [<Question: What's up?>]>

  # Django提供了丰富的数据库查询API，通过关键字来查询。
  >>> Question.objects.filter(id=1)
  <QuerySet [<Question: What's up?>]>
  >>> Question.objects.filter(question_text__startswith='What')
  <QuerySet [<Question: What's up?>]>

  # 获取一个今年发布的Question
  >>> from django.utils import timezone
  >>> current_year = timezone.now().year
  >>> Question.objects.get(pub_date__year=current_year)
  <Question: What's up?>

  # 回应的ID不存在，会抛出一个错误
  >>> Question.objects.get(id=2)
  Traceback (most recent call last):
      ...
  DoesNotExist: Question matching query does not exist.

  # 通过主键来查询是一件平常事，
  # 因此Django提供了主键查询的快捷方式
  # 下面等价于Question.objects.get(id=1)
  >>> Question.objects.get(pk=1)
  <Question: What's up?>

  # 确认一下自定义方法是否工作正常
  >>> q = Question.objects.get(pk=1)
  >>> q.was_published_recently()
  True

  # Give the Question a couple of Choices. The create call constructs a new
  # Choice object, does the INSERT statement, adds the choice to the set
  # of available choices and returns the new Choice object. Django creates
  # a set to hold the "other side" of a ForeignKey relation
  # (e.g. a question's choice) which can be accessed via the API.
  # TODO: 这段不是很理解。
  >>> q = Question.objects.get(pk=1)

  # 显示所有相关的Choices
  >>> q.choice_set.all()
  <QuerySet []>

  # 新建3个Choices
  >>> q.choice_set.create(choice_text='Not much', votes=0)
  <Choice: Not much>
  >>> q.choice_set.create(choice_text='The sky', votes=0)
  <Choice: The sky>
  >>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

  # Choice object可以通过API来访问与它关联的Question objects
  >>> c.question
  <Question: What's up?>

  # And vice versa: Question objects get access to Choice objects.
  >>> q.choice_set.all()
  <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
  >>> q.choice_set.count()
  3

  # The API automatically follows relationships as far as you need.
  # Use double underscores to separate relationships.
  # This works as many levels deep as you want; there's no limit.
  # Find all Choices for any question whose pub_date is in this year
  # (reusing the 'current_year' variable we created above).
  >>> Choice.objects.filter(question__pub_date__year=current_year)
  <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

  # Let's delete one of the choices. Use delete() for that.
  >>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
  >>> c.delete()
