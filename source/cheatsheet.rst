.. _cheat-sheet:

******************
Sphinx备忘单
******************

这是一份奇妙的便签,关于你在Sphinx和ReST中需要用到的常见资料。你可以查看 :ref:`cheatsheet-literal`。


.. _formatting-text:

文本格式
===============
你可以使用行间标记来标记文本成 *斜体*， **粗体**，或者 ``monotype``。

你可以相当简单地展现代码块::

   import numpy as np
   x = np.random.rand(12)

或者之间将代码纳入进来:

.. literalinclude:: ellipses.py


.. _making-a-list:

制作列表
=============
生成列表是一件非常容易的事情。

要点列表
-------------
这段介绍如何制作要点列表。

* point A

* point B

* point C

有序列表
------------------
这段介绍何如制作有序列表。

#. point A

#. point B

#. point C


.. _making-a-table:

制作表格
==============
这里介绍了怎么样制作一张表格——如果你指向制作一张列表请看 :ref:`making-a-list` 。

==================   ============
Name                 Age
==================   ============
John D Hunter        40
Cast of Thousands    41
And Still More       42
==================   ============

脚注
=============
这段介绍一个脚注是如何制作的。[1]_ 如下代码，效果见脚注[1]::

    Lorem ipsum [#f1]_ dolor sit amet ... [#f2]_

    .. rubric:: Footnotes

    .. [#f1] Text of the first footnote.
    .. [#f2] Text of the second footnote.

.. _making-links:

制作超链接
============
你可以很容易的生成一个超链接到 `新浪 <http://sina.com.cn>`_ 或者到这篇文档的一个
小结 （见： :ref:`making-a-table` ） 又或者到另外一个文档。

（未翻译）You can also reference classes, modules, functions, etc that are
documented using the sphinx `autodoc
<http://sphinx.pocoo.org/ext/autodoc.html>`_ facilites.  For example,
see the module :mod:`matplotlib.backend_bases` documentation, or the
class :class:`~matplotlib.backend_bases.LocationEvent`, or the method
:meth:`~matplotlib.backend_bases.FigureCanvasBase.mpl_connect`.

.. _cheatsheet-literal:

这个文档的源码
==================

.. literalinclude:: cheatsheet.rst

英文源码 [2]_
==================
.. literalinclude:: cheatsheet_eng_version.txt

.. rubric:: 脚注
.. [1] 这是一个脚注的内容。
.. [2] 英文源链接：http://matplotlib.org/sampledoc/cheatsheet.html#making-a-list
