.. module:: bottle

.. _paste: http://pythonpaste.org/modules/evalexception.html
.. _pylons: http://pylonshq.com/
.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/

==========================
常见问题
==========================

关于Bottle
============

Bottle适合用于复杂的应用吗？
---------------------------------------------

Bottle是一个 *迷你* 框架，被设计来提供原型开发和创建小的Web应用和服务。它能快速上手，并帮助你快速完成任务，但缺少一些高级功能和一些其他框架已提供的已知问题解决方法(例如:MVC， ORM，表单验证，手脚架，XML-RPC)。尽管 *可以* 添加这些功能，然后通过Bottle来开发复杂的应用，但我们还是建议你使用一些功能完备的Web框架，例如 pylons_ 或 paste_ 。


常见的，意料之外的问题
============================


"Template Not Found" in mod_wsgi/mod_python
--------------------------------------------------------------------------------

Bottle会在 ``./`` 和 ``./views/`` 目录中搜索模板。在一个 mod_python_ 或 mod_wsgi_ 环境，当前工作目录( ``./`` )是由Apache的设置决定的。你应该在模板的搜索路径中添加一个绝对路径。

::

    bottle.TEMPLATE_PATH.insert(0,'/absolut/path/to/templates/')

这样，Bottle就能在正确的目录下搜索模板了

动态route和反斜杠
--------------------------------------------------------------------------------

在 :ref:`dynamic route syntax <tutorial-dynamic-routes>` 中， ``:name`` 匹配任何字符，直到出现一个反斜杠。工作方式相当与 ``[^/]+`` 这样一个正则表达式。为了将反斜杠包涵进来，你必须在 ``:name`` 中添加一个自定义的正则表达式。例如： ``/images/:filepath#.*#`` 会匹配 ``/images/icons/error.png`` ，但不匹配 ``/images/:filename`` 。

反向代理的问题
--------------------------------------------------------------------------------

只用Bottle知道公共地址和应用位置的情况下，重定向和url-building才会起作用。
(译者注：保留原文)
Redirects and url-building only works if bottle knows the public address and location of your application.
如果Bottle应用运行在反向代理或负载均衡后面，一些信息也许会丢失。例如， ``wsgi.url_scheme`` 的值或 ``Host`` 头或许只能获取到代理的请求信息，而不是真实的用户请求。下面是一个简单的中间件代码片段，处理上面的问题，获取正确的值。

::

  def fix_environ_middleware(app):
    def fixed_app(environ, start_response):
      environ['wsgi.url_scheme'] = 'https'
      environ['HTTP_X_FORWARDED_HOST'] = 'example.com'
      return app(environ, start_response)
    return https_app

  app = bottle.default_app()    
  app.wsgi = fix_environ_middleware(app.wsgi)
  

