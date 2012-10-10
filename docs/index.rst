.. highlight:: python
.. currentmodule:: bottle

.. _mako: http://www.makotemplates.org/
.. _cheetah: http://www.cheetahtemplate.org/
.. _jinja2: http://jinja.pocoo.org/2/
.. _paste: http://pythonpaste.org/
.. _fapws3: https://github.com/william-os4y/fapws3
.. _bjoern: https://github.com/jonashaag/bjoern
.. _flup: http://trac.saddi.com/flup
.. _cherrypy: http://www.cherrypy.org/
.. _WSGI: http://www.wsgi.org/wsgi/
.. _Python: http://python.org/
.. _testing: https://github.com/defnull/bottle/raw/master/bottle.py
.. _issue_tracker: https://github.com/defnull/bottle/issues
.. _PyPi: http://pypi.python.org/pypi/bottle
.. _Python标准库: http://docs.python.org/library

============================
Bottle: Python Web Framework
============================

Bottle是一个快速，简单，轻量级的 Python_ WSGI_ Web框架。单一文件，只依赖 Python标准库_ 。

* **URL映射(Routing):** 将URL请求映射到Python函数，使URL更简洁。
* **模板(Templates):** 快速且pythonic的 :ref:`内置模板引擎 <tutorial-templates>` ，同时支持 mako_, jinja2_ 和 cheetah_ 等模板。
* **基础功能(Utilities):** 方便地访问表单数据，上传文件，使用cookie，查看HTTP元数据。
* **开发服务器(Server):** 内置了开发服务器，且支持 paste_, fapws3_ , bjoern_, `Google App Engine <http://code.google.com/intl/en-US/appengine/>`_, cherrypy_ 等符合 WSGI_ 标准的HTTP服务器。

.. rubric:: 示例: "Hello World"

::

  from bottle import route, run

  @route('/hello/:name')
  def index(name='World'):
      return '<b>Hello %s!</b>' % name

  run(host='localhost', port=8080)

将其保存为py文件并执行，用浏览器访问 `<http://localhost:8080/hello/world>`_ 即可看到效果。就这么简单！

.. rubric:: 下载和安装

.. _download:

.. __: https://github.com/defnull/bottle/raw/master/bottle.py

通过 PyPi_ (``easy_install -U bottle``)安装最新的稳定版，或下载 `bottle.py`__ (开发版)到你的项目文件夹。 Bottle除了Python标准库无任何依赖 [1]_ ，同时支持 **Python 2.5+ and 3.x** 。

用户指南
===============
如果你有将Bottle用于Web开发的打算，请继续看下去。如果这份文档没有解决你遇到的问题，尽管在 `邮件列表 <mailto:bottlepy@googlegroups.com>`_ 中吼出来吧（译者注：用英文哦）。

.. toctree::
   :maxdepth: 2

   tutorial
   routing
   stpl
   api
   plugins/index


知识库
==============

收集文章，使用指南和HOWTO

.. toctree::
   :maxdepth: 2

   tutorial_app
   async
   recipes
   faq


开发和贡献
============================

这些章节是为那些对Bottle的开发和发布流程感兴趣的开发者准备的。

.. toctree::
   :maxdepth: 2

   changelog
   development
   plugindev


.. toctree::
   :hidden:

   plugins/index
   
许可证
==================

代码和文件皆使用MIT许可证:

.. include:: ../LICENSE
  :literal:

然而，许可证 *不包含* Bottle的logo。logo用作指向Bottle主页的连接，或未修改过的类库。

如要用于其它用途，请先请求许可。


.. rubric:: Footnotes

.. [1] 如果使用了第三方的模板或HTTP服务器，则需要安装相应的第三方模块。

