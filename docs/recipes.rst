.. module:: bottle

.. _beaker: http://beaker.groovie.org/
.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _werkzeug: http://werkzeug.pocoo.org/documentation/dev/debug.html
.. _paste: http://pythonpaste.org/modules/evalexception.html
.. _pylons: http://pylonshq.com/
.. _gevent: http://www.gevent.org/
.. _compression: https://github.com/defnull/bottle/issues/92
.. _GzipFilter: http://www.cherrypy.org/wiki/GzipFilter
.. _cherrypy: http://www.cherrypy.org
.. _heroku: http://heroku.com

秘诀
=============

这里收集了一些常见用例的代码片段和例子.

使用Session
----------------------------

Bottle自身并没有提供Session的支持，因为在一个迷你框架里面，没有合适的方法来实现。根据需求和使用环境，你可以使用 beaker_ 中间件或自己来实现。
下面是一个使用beaker的例子，Session数据存放在"./data"目录里面。

::

    import bottle
    from beaker.middleware import SessionMiddleware

    session_opts = {
        'session.type': 'file',
        'session.cookie_expires': 300,
        'session.data_dir': './data',
        'session.auto': True
    }
    app = SessionMiddleware(bottle.app(), session_opts)

    @bottle.route('/test')
    def test():
      s = bottle.request.environ.get('beaker.session')
      s['test'] = s.get('test',0) + 1
      s.save()
      return 'Test counter: %d' % s['test']

    bottle.run(app=app)

Debugging with Style: 调试中间件
--------------------------------------------------------------------------------

Bottle捕获所有应用抛出的异常，防止异常导致WSGI服务器崩溃。如果内置的 :func:`debug` 模式不能满足你的要求，你想在你自己写的中间件里面处理这些异常，那么你可以关闭这个功能。

::

    import bottle
    app = bottle.app() 
    app.catchall = False #现在，Bottle会重新抛出所有捕获到的异常
    myapp = DebuggingMiddleware(app) #这里替换成你的中间件
    bottle.run(app=myapp)

现在，Bottle仅会捕获并处理它自己抛出的异常( :exc:`HTTPError` , :exc:`HTTPResponse` 和 :exc:`BottleException` )，你的中间件可以处理剩下的那些异常。

werkzeug_ 和 paste_ 这两个第三方库都提供了非常强大的调试中间件。如果是 werkzeug_ ，可看看 :class:`werkzeug.debug.DebuggedApplication` ，如果是 paste_ ，可看看 :class:`paste.evalexception.middleware.EvalException` 。它们都可让你检查运行栈，甚至在保持运行栈上下文的情况下，执行Python代码。所以 **不要在生产环境中使用它们** 。


Unit-Testing
--------------------------------------------------------------------------------

Unit测试一般用于测试应用中的函数，但不需要一个WSGI环境。

使用 `Nose <http://readthedocs.org/docs/nose>`_ 的简单例子。

::

    import bottle
    
    @bottle.route('/')
    def index():
        return 'Hi!'

    if __name__ == '__main__':
        bottle.run()

测试代码::

    import mywebapp
    
    def test_webapp_index():
        assert mywebapp.index() == 'Hi!'

在这个例子中，Bottle的route()函数没有被执行，仅测试了index()函数。


Functional Testing
--------------------------------------------------------------------------------

任何基于HTTP的测试系统都可用于测试WSGI服务器，但是有些测试框架与WSGI服务器工作得更好。它们可以在一个可控环境里运行WSGI应用，充分利用traceback和调试工具。 `Testing tools for WSGI <http://www.wsgi.org/en/latest/testing.html>`_ 是一个很好的上手工具。

使用 `WebTest <http://webtest.pythonpaste.org/>`_ 和 `Nose <http://readthedocs.org/docs/nose>`_ 的例子。

::

    from webtest import TestApp
    import mywebapp

    def test_functional_login_logout():
        app = TestApp(mywebapp.app)
        
        app.post('/login', {'user': 'foo', 'pass': 'bar'}) # 登录， 获取一个cookie

        assert app.get('/admin').status == '200 OK'        # 成功抓取一个页面

        app.get('/logout')                                 # 登出
        app.reset()                                        # 丢弃cookie

        # 抓取同一个页面，失败！
        assert app.get('/admin').status == '401 Unauthorized'


嵌入其他WSGI应用
--------------------------------------------------------------------------------

并不建议你使用这个方法，你应该在Bottle前面使用一个中间件来做这样的事情。但你确实可以在Bottle里面调用其他WSGI应用，让Bottle扮演一个中间件的角色。下面是一个例子。

::

    from bottle import request, response, route
    subproject = SomeWSGIApplication()

    @route('/subproject/:subpath#.*#', method='ALL')
    def call_wsgi(subpath):
        new_environ = request.environ.copy()
        new_environ['SCRIPT_NAME'] = new_environ.get('SCRIPT_NAME','') + '/subproject'
        new_environ['PATH_INFO'] = '/' + subpath
        def start_response(status, headerlist):
            response.status = int(status.split()[0])
            for key, value in headerlist:
                response.add_header(key, value)
        return app(new_environ, start_response)

再次强调，并不建议使用这种方法。之所以介绍这种方法，是因为很多人问起，如何在Bottle中调用WSGI应用。


忽略尾部的反斜杠
--------------------------------------------------------------------------------

在Bottle看来， ``/example`` 和 ``/example/`` 是两个不同的route [1]_ 。为了一致对待这两个URL，你应该添加两个route。

::

    @route('/test')
    @route('/test/')
    def test(): return '反斜杠? 不?'

或者添加一个WSGI中间件来处理这种情况。

::

    class StripPathMiddleware(object):
      def __init__(self, app):
        self.app = app
      def __call__(self, e, h):
        e['PATH_INFO'] = e['PATH_INFO'].rstrip('/')
        return self.app(e,h)

    app = bottle.app()
    myapp = StripPathMiddleware(app)
    bottle.run(app=myapp)

.. rubric:: Footnotes

.. [1] 因为确实如此，见 <http://www.ietf.org/rfc/rfc3986.txt>


Keep-alive 请求
-------------------

.. note::

    详见 :doc:`async` 。

像XHR这样的"push"机制，需要在HTTP响应头中加入 "Connection: keep-alive" ，以便在不关闭连接的情况下，写入响应数据。WSGI并不支持这种行为，但如果在Bottle中使用 gevent_ 这个异步框架，还是可以实现的。下面是一个例子，可配合 gevent_ HTTP服务器或 paste_ HTTP服务器使用(也许支持其他服务器，但是我没试过)。在run()函数里面使用 ``server='gevent'`` 或 ``server='paste'`` 即可使用这两种服务器。

::

    from gevent import monkey; monkey.patch_all()

    import time
    from bottle import route, run
    
    @route('/stream')
    def stream():
        yield 'START'
        time.sleep(3)
        yield 'MIDDLE'
        time.sleep(5)
        yield 'END'
    
    run(host='0.0.0.0', port=8080, server='gevent')

通过浏览器访问 ``http://localhost:8080/stream`` ，可看到'START'，'MIDDLE'，和'END'这三个字眼依次出现，一共用了8秒。

Gzip压缩
--------------------------

.. note::
   详见 compression_

Gzip压缩，可加速网站静态资源(例如CSS和JS文件)的访问。
人们希望Bottle支持Gzip压缩，（但是不支持)......

支持Gzip压缩并不简单，一个合适的Gzip实现应该满足以下条件。

* 压缩速度要快
* 如果浏览器不支持，则不压缩
* 不压缩那些已经充分压缩的文件(图像，视频)
* 不压缩动态文件
* 支持两种压缩算法(gzip和deflate)
* 缓存那些不经常变化的压缩文件
* 不验证缓存中那些已经变化的文件(De-validate the cache if one of the files changed anyway)
* 确保缓存不太大
* 不缓存小文件，因为寻道时间或许比压缩时间还长

因为有上述种种限制，建议由WSGI服务器来处理Gzip压缩而不是Bottle。像 cherrypy_ 就提供了一个 GzipFilter_ 中间件来处理Gzip压缩。


使用钩子
----------------------

例如，你想提供跨域资源共享，可参考下面的例子。

::

    from bottle import hook, response, route

    @hook('after_request')
    def enable_cors():
        response.headers['Access-Control-Allow-Origin'] = '*'

    @route('/foo')
    def say_foo():
        return 'foo!'

    @route('/bar')
    def say_bar():
        return {'type': 'friendly', 'content': 'Hi!'}

你也可以使用 ``before_callback`` ，这样在route的回调函数被调用之前，都会调用你的钩子函数。

在Heroku中使用Bottle
------------------------

Heroku_ ，一个流行的云应用平台，提供Python支持。

这份教程基于 `Heroku Quickstart 
<http://devcenter.heroku.com/articles/quickstart>`_, 
用Bottle特有的代码替换了 `Getting Started with Python on Heroku/Cedar 
<http://devcenter.heroku.com/articles/python>`_ 中的
`Write Your App <http://devcenter.heroku.com/articles/python#write_your_app>`_ 这部分。

::

    import os
    from bottle import route, run

    @route("/")
    def hello_world():
            return "Hello World!"

    run(host="0.0.0.0", port=int(os.environ.get("PORT", 5000)))

Heroku使用 `os.environ` 字典来提供Bottle应用需要监听的端口。
