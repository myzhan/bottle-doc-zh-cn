.. _flup: http://trac.saddi.com/flup
.. _gae: http://code.google.com/appengine/docs/python/overview.html
.. _wsgiref: http://docs.python.org/library/wsgiref.html
.. _cherrypy: http://www.cherrypy.org/
.. _paste: http://pythonpaste.org/
.. _rocket: http://pypi.python.org/pypi/rocket
.. _gunicorn: http://pypi.python.org/pypi/gunicorn
.. _fapws3: http://www.fapws.org/
.. _tornado: http://www.tornadoweb.org/
.. _twisted: http://twistedmatrix.com/
.. _diesel: http://dieselweb.org/
.. _meinheld: http://pypi.python.org/pypi/meinheld
.. _bjoern: http://pypi.python.org/pypi/bjoern
.. _gevent: http://www.gevent.org/
.. _eventlet: http://eventlet.net/
.. _waitress: http://readthedocs.org/docs/waitress/en/latest/

.. _tutorial-deployment:

================================================================================
部署
================================================================================

不添加任何参数，直接运行Bottle的 :func:`run` 函数，会启动一个本地的开发服务器，监听8080端口。你可在同一部主机上访问http://localhost:8080/来测试你的应用。

可更改服务器监听的IP地址(例如： ``run(host='192.168.0.1')`` )，来让应用对其可访问，或者让服务器接受所有地址的请求(例如： ``run(host='0.0.0.0')`` )。可通过类似的方法改变服务器监听的端口，但如果你选择了一个小于1024的端口，则需要root权限。HTTP服务器的标准端口是80端口。

::

  run(host='0.0.0.0', port=80) # 监听所有IP地址的HTTP请求

可选服务器
================================================================================

内置的服务器基于 `wsgiref WSGIServer <http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server>`_ 。这个单线程的HTTP服务器很适合用于开发，但当服务器的负载上升的时候，会成为一个性能瓶颈。有三种方法可消除这一瓶颈：

* 使用多线程或异步的服务器
* 运行多个服务器，使用负载均衡
* 同时使用上面两种方法

**多线程** 服务器是经典的解决方法。它们非常鲁棒(稳定)，快速和易于管理。但它们也有缺点，同时只能处理固定数量的请求，因为全局解释器锁(GIL)的存在，所有只能利用一个CPU核心。对于大部分的应用而言，这无伤大雅，也许它们大部分时间都在等待网络IO。但对于一些比较耗CPU的应用(例如图像处理)，就要考虑性能问题了。

**异步** 服务器非常快，同时处理的请求数无上限，也方便管理。为了充分利用它们的潜力，你需要根据异步的原理来设计你的应用，了解它们的工作机制。

**多进程** (forking) 服务器就没有受到GIL的限制，能利用多个CPU核心，但服务器实例之间的交流代价比较高昂。你需要一个数据库或消息队列来在进程之间共享状态，或将你的应用设计成根本不需要共享状态。多进程服务器的安装也比较负责，但已经有很多好的教程了。

更改服务器后端
================================================================================

提高性能最简单的办法，是安装一个多线程的服务器库，例如 paste_ 和 cherrypy_ ，然后告诉Bottle使用它们。

::

    bottle.run(server='paste')

Bottle为很多常见的WSGI服务器都编写了适配器，能自动化安装过程。下面是一个不完整的清单:

========  ============  ======================================================
名称       主页           描述
========  ============  ======================================================
cgi                     Run as CGI script
flup      flup_         Run as FastCGI process
gae       gae_          用于Google App Engine
wsgiref   wsgiref_      默认的单线程服务器
cherrypy  cherrypy_     多线程，稳定
paste     paste_        多线程，稳定，久经考验，充分测试
rocket    rocket_       多线程
waitress  waitress_     多线程，源于Pyramid
gunicorn  gunicorn_     Pre-forked, partly written in C
eventlet  eventlet_     支持WSGI的异步框架
gevent    gevent_       异步 (greenlets)
diesel    diesel_       异步 (greenlets)
fapws3    fapws3_       异步 (network side only), written in C
tornado   tornado_      异步, powers some parts of Facebook
twisted   twisted_      异步, well tested but... twisted
meinheld  meinheld_     异步, partly written in C
bjoern    bjoern_       异步, very fast and written in C
auto                    自动选择一个服务器
========  ============  ======================================================

完整的列表在 :data:`server_names`.

如果没有适合你的服务器的适配器，或者你需要更多地控制服务器的安装，你也许需要手动启动服务器。可参考你的服务器的文档，看看是如何运行一个WSGI应用。下面是一个例子。

::

    application = bottle.default_app()
    from paste import httpserver
    httpserver.serve(application, host='0.0.0.0', port=80)


Apache mod_wsgi
--------------------------------------------------------------------------------

除了直接在Bottle里面运行HTTP服务器，你也可以将你的应用部署到一个Apache服务器上，使用 mod_wsgi_ 来运行。

你需要的只是提供一个 ``application`` 对象的 ``app.wsgi`` 文件。mod_wsgi会使用这个对象来启动你的应用，这个对象必须是兼容WSGI的 callable对象。

 ``/var/www/yourapp/app.wsgi`` 文件 
 
::

    # 更改工作目录，相关的路径(例如查找模板的路径)又有效了
    os.chdir(os.path.dirname(__file__))
    
    import bottle
    # 创建或导入你的Bottle应用
    # 不要使用bottle.run函数
    application = bottle.default_app()

Apache的配置

::

    <VirtualHost *>
        ServerName example.com
        
        WSGIDaemonProcess yourapp user=www-data group=www-data processes=1 threads=5
        WSGIScriptAlias / /var/www/yourapp/app.wsgi
        
        <Directory /var/www/yourapp>
            WSGIProcessGroup yourapp
            WSGIApplicationGroup %{GLOBAL}
            Order deny,allow
            Allow from all
        </Directory>
    </VirtualHost>



Google AppEngine
--------------------------------------------------------------------------------

.. versionadded:: 0.9

``gae`` 这个服务器适配器可在Google App Engine上运行你的应用。它的工作方式和 ``cgi`` 适配器类似，并没有启动一个HTTP服务器，只是针对做了一下准备和优化工作。确保你的应用遵守Google App Engine的API。

::

    bottle.run(server='gae') # 无需host和port参数

直接让GAE来提供静态文件服务总是一个好主意。类似的可工作的 ``app.yaml`` 文件如下。

::

    application: myapp
    version: 1
    runtime: python
    api_version: 1

    handlers:
    - url: /static
      static_dir: static

    - url: /.*
      script: myapp.py


负载均衡 (手动安装)
--------------------------------------------------------------------------------

单一的Python进程一次只能使用一个CPU内核，即使CPU是多核的。我们的方法就是在多核CPU的机器上，使用多线程来实现负载均衡。

不只是启动一个应用服务器，你需要同时启动多个应用服务器，监听不同的端口(localhost:8080, 8081, 8082, ...)。你可选择任何服务器，甚至那些异步服务器。然后一个高性能的负载均衡器，像一个反向代理那样工作，将新的请求发送到一个随机端口，在多个服务器之间分散压力。这样你就可以利用所有的CPU核心，甚至在多个机器上实现负载均衡。

 Pound_ 是最快的负载均衡器之一，但是只要有代理模块的Web服务器，也可以充当这一角色。

Pound的例子

::

    ListenHTTP
        Address 0.0.0.0
        Port    80

        Service
            BackEnd
                Address 127.0.0.1
                Port    8080
            End
            BackEnd
                Address 127.0.0.1
                Port    8081
            End
        End
    End

Apache的例子

::

    <Proxy balancer://mycluster>
    BalancerMember http://192.168.1.50:80
    BalancerMember http://192.168.1.51:80
    </Proxy>
    ProxyPass / balancer://mycluster 

Lighttpd的例子

::

    server.modules += ( "mod_proxy" )
    proxy.server = (
        "" => (
            "wsgi1" => ( "host" => "127.0.0.1", "port" => 8080 ),
            "wsgi2" => ( "host" => "127.0.0.1", "port" => 8081 )
        )
    )


CGI这个老好人
================================================================================

CGI服务器会为每个请求启动一个进程。虽然这样代价高昂，但有时这是唯一的选择。 `cgi` 这个适配器实际上并没有启动一个CGI服务器，只是将你的Bottle应用转换成了一个有效的CGI应用。

::

    bottle.run(server='cgi')



