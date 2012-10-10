.. module:: bottle

.. _Apache Server:
.. _Apache: http://www.apache.org/
.. _cherrypy: http://www.cherrypy.org/
.. _decorator: http://docs.python.org/glossary.html#term-decorator
.. _flup: http://trac.saddi.com/flup
.. _http_code: http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
.. _http_method: http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html
.. _json: http://de.wikipedia.org/wiki/JavaScript_Object_Notation
.. _lighttpd: http://www.lighttpd.net/
.. _mako: http://www.makotemplates.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _Paste: http://pythonpaste.org/
.. _Pound: http://www.apsis.ch/pound/
.. _`WSGI Specification`: http://www.wsgi.org/wsgi/
.. _issue: http://github.com/defnull/bottle/issues
.. _Python: http://python.org/
.. _SimpleCookie: http://docs.python.org/library/cookie.html#morsel-objects
.. _testing: http://github.com/defnull/bottle/raw/master/bottle.py

========
教程
========

这份教程将向你介绍Bottle的开发理念和功能特性。既介绍Bottle的基本用法，也包含了进阶用法。你可以从头到尾通读一遍，也可当做开发时的参考。你也许对自动生成的 :doc:`api` 感兴趣。它包含了更多的细节，但解释没有这份教程详细。在 :doc:`recipes` 或 :doc:`faq` 可找到常见问题的解决办法。如果需要任何帮助，可加入我们的 `邮件列表 <mailto:bottlepy@googlegroups.com>`_ 或在 `IRC频道 <http://webchat.freenode.net/?channels=bottlepy>`_ 和我们交流。

.. _installation:

安装
==============================================================================

.. __: https://github.com/defnull/bottle/raw/master/bottle.py

Bottle不依赖其他库，你需要做的仅是下载 `bottle.py`__ (开发版)到你的项目文件夹，然后开始写代码。

.. code-block:: bash

    $ wget http://bottlepy.org/bottle.py

在终端运行以上命令，即可下载到Bottle的最新开发版，包含了所有新功能特性。如果更需要稳定性，你应该坚持使用Bottle的稳定版本。可在 `PyPi <http://pypi.python.org/pypi/bottle>`_ 下载稳定版本，然后通过 :command:`pip` (推荐), :command:`easy_install` 或你的包管理软件安装。

.. code-block:: bash

    $ sudo pip install bottle              # 推荐
    $ sudo easy_install bottle             # 若无pip，可尝试这个
    $ sudo apt-get install python-bottle   # 适用于debian, ubuntu, ...

你需要Python 2.5或以上版本（包括3.x）来运行bottle应用。如果你没有管理员权限或你不想将Bottle安装为整个系统范围内可用，可先创建一个 `virtualenv <http://pypi.python.org/pypi/virtualenv>`_ :

.. code-block:: bash

    $ virtualenv develop              # 创建一个虚拟环境
    $ source develop/bin/activate     # 使用虚拟环境里的Python解析器
    (develop)$ pip install -U bottle  # 在虚拟环境中安装Bottle

如果还未安装virtualenv:

.. code-block:: bash

    $ wget https://raw.github.com/pypa/virtualenv/master/virtualenv.py
    $ python virtualenv.py develop    # 创建一个虚拟环境
    $ source develop/bin/activate     # 使用虚拟环境里的Python解析器
    (develop)$ pip install -U bottle  # 在虚拟环境中安装Bottle



Quickstart: "Hello World"
==============================================================================

到目前为止，我假设你已经 :ref:`安装 <installation>` 好了bottle或已将bottle.py拷贝到你的项目文件夹。接下来我们就可以写一个非常简单的"Hello World"了::

    from bottle import route, run

    @route('/hello')
    def hello():
        return "Hello World!"

    run(host='localhost', port=8080, debug=True)

就这么简单！保存为py文件并执行，用浏览器访问 http://localhost:8080/hello 就可以看到"Hello World!"。它的执行流程大致如下:

:func:`route` 函数将一段代码绑定到一个URL，在这个例子中, 我们将 ``hello()`` 函数绑定给了 ``/hello`` 。我们称之为 `route` (也是该修饰器的函数名) ，这是Bottle框架最重要的开发理念。你可以根据需要定义任意多的 `route` 。在浏览器请求一个URL的时候，框架自动调用与之相应的函数，接着将函数的返回值发送给浏览器。就这么简单！

最后一行调用的 :func:`run` 函数启动了内置的开发服务器。它监听 `localhost` 的8080端口并响应请求， :kbd:`Control-c` 可将其关闭。到目前为止，这个内置的开发服务器已经足够用于日常的开发测试了。它根本不需要安装，就可以让你的应用跑起来。在教程的后面，你将学会如何让你的应用跑在其他服务器上面（译者注：内置服务器不能满足生产环境的要求）

:ref:`tutorial-debugging` 在早期开发的时候非常有用，但请务必记得，在生产环境中将其关闭。

毫无疑问，这是一个十分简单例子，但它展示了用Bottle做应用开发的基本理念。接下来你将了解到其他开发方式。

.. _tutorial-default:

`默认应用`
------------------------------------------------------------------------------

基于简单性考虑，这份教程中的大部分例子都使用一个模块层面的 :func:`route` 修饰器函数来定义route。这样的话，所有route都添加到了一个全局的“默认应用”里面，即是在第一次调用 :func:`route` 函数时，创建的一个 :class:`Bottle` 类的实例。其他几个模块层面的修饰器函数都与这个“默认应用”有关，如果你偏向于面向对象的做法且不介意多打点字，你可以创建一个独立的应用对象，这样就可避免使用全局范围的“默认应用”。

::

    from bottle import Bottle, run

    app = Bottle()

    @app.route('/hello')
    def hello():
        return "Hello World!"

    run(app, host='localhost', port=8080)

接下来的 :ref:`default-app` 章节中将更详细地介绍这种做法。现在，你只需知道不止有一种选择就好了。

.. _tutorial-routing:

URL映射
==============================================================================

在上一章中，我们实现了一个十分简单的web应用，只有一个URL映射(route)。让我们再来看一下“Hello World”中与routing有关的部分。

::

    @route('/hello')
    def hello():
        return "Hello World!"

:func:`route` 函数将一个URL路径与一个回调函数关联了起来，然后在 :ref:`default application <tutorial-default>` 中添加了一个URL映射(route)。但只有一个route的应用未免太无聊了，让我们试着再添加几个route吧。::

    @route('/')
    @route('/hello/<name>')
    def greet(name='Stranger'):
        return 'Hello %s, how are you?' % name

这个例子说明了两件事情，一个回调函数可绑定多个route，你也可以在URL中添加通配符，然后在回调函数中使用它们。

.. _tutorial-dynamic-routes:

动态URL映射
-------------------------------------------------------------------------------

包含通配符的route，我们称之为动态route(与之对应的是静态route)，它能匹配多个URL地址。一个通配符包含在一对尖括号里面(像这样 ``<name>`` )，通配符之间用"/"分隔开来。如果我们将URL定义为 ``/hello/<name>`` 这样，那么它就能匹配 ``/hello/alice`` 和 ``/hello/bob`` 这样的浏览器请求，但不能匹配 ``/hello`` , ``/hello/`` 和 ``/hello/mr/smith`` 。

URL中的通配符都会当作参数传给回调函数，直接在回调函数中使用。这样可以漂亮地实现RESTful形式的URL。例子如下::

    @route('/wiki/<pagename>')            # 匹配 /wiki/Learning_Python
    def show_wiki_page(pagename):
        ...

    @route('/<action>/<user>')            # 匹配 /follow/defnull
    def user_api(action, user):
        ...

从0.10版本开始，过滤器(Filter)可被用来定义特殊类型的通配符，在传通配符给回调函数之前，先自动转换通配符类型。包含过滤器的通配符定义一般像 ``<name:filter>`` 或 ``<name:filter:config>`` 这样。config部分是可选的，其语法由你使用的过滤器决定。

已实现下面几种形式的过滤器，后续可能会继续添加:

* **:int** 匹配一个数字，自动将其转换为int类型。
* **:float** 与:int类似，用于浮点数。
* **:path** 匹配一个路径(包含"/")
* **:re** 匹配config部分的一个正则表达式，不更改被匹配到的值

让我们来看看具体的使用例子

::

    @route('/object/<id:int>')
    def callback(id):
        assert isinstance(id, int)

    @route('/show/<name:re:[a-z]+>')
    def callback(name):
        assert name.isalpha()

    @route('/static/<path:path>')
    def callback(path):
        return static_file(path, ...)

你可以添加自己的过滤器。详见 :doc:`routing` 。
（译者注：参考Bottle的Router类，源码300行左右）

从 **Bottle 0.10** 版本开始，可以用新的语法来在URL中定义通配符，更简单了。新旧语法之间的对比如下:

=================== ====================
旧                    新
=================== ====================
``:name``           ``<name>``
``:name#regexp#``   ``<name:re:regexp>``
``:#regexp#``       ``<:re:regexp>``
``:##``             ``<:re>``
=================== ====================

请尽可能在新项目中使用新的语法。虽然现在依然兼容旧语法，但终究会将其废弃的。

HTTP请求方法
------------------------------------------------------------------------------

.. __: http_method_

HTTP协议定义了多个 `请求方法`__  来满足不同的需求。所有route默认使用GET方法，只响应GET请求。可给 :func:`route` 函数指定 ``method`` 参数。或用  :func:`get` ， :func:`post` ， :func:`put`  或 :func:`delete` 等函数来代替 :func:`route` 函数。

POST方法一般用于HTML表单的提交。下面是一个使用POST来实现用户登录的例子::

    from bottle import get, post, request

    @get('/login') # 或 @route('/login')
    def login_form():
        return '''<form method="POST" action="/login">
                    <input name="name"     type="text" />
                    <input name="password" type="password" />
                    <input type="submit" />
                  </form>'''

    @post('/login') # 或 @route('/login', method='POST')
    def login_submit():
        name     = request.forms.get('name')
        password = request.forms.get('password')
        if check_login(name, password):
            return "<p>登录成功</p>"
        else:
            return "<p>登录失败</p>"
            
在这个例子中， ``/login`` 绑定了两个回调函数，一个回调函数响应GET请求，一个回调函数响应POST请求。如果浏览器使用GET请求访问 ``/login`` ，则调用login_form()函数来返回登录页面，浏览器使用POST方法提交表单后，调用login_submit()函数来检查用户有效性，并返回登录结果。接下来的 :ref:`tutorial-request` 章节中，会详细介绍 :attr:`Request.forms` 的用法。

.. rubric:: 特殊请求方法: HEAD 和 ANY

HEAD方法类似于GET方法，但服务器不会返回HTTP响应正文，一般用于获取HTTP原数据而不用下载整个页面。Bottle像处理GET请求那样处理HEAD请求，但是会自动去掉HTTP响应正文。你无需亲自处理HEAD请求。

另外，非标准的ANY方法做为一个低优先级的fallback：在没有其它route的时候，监听ANY方法的route会匹配所有请求，而不管请求的方法是什么。这对于用做代理的route很有用，可将所有请求都重定向给子应用。

总而言之：HEAD请求被响应GET请求的route来处理，响应ANY请求的route处理所有请求，但仅限于没有其它route来匹配原先的请求的情况。就这么简单。


静态文件映射
------------------------------------------------------------------------------

Bottle不会处理像图片或CSS文件的静态文件请求。你需要给静态文件提供一个route，一个回调函数(用于查找和控制静态文件的访问)。

::

  from bottle import static_file
  @route('/static/<filename>')
  def server_static(filename):
      return static_file(filename, root='/path/to/your/static/files')

 :func:`static_file` 函数用于响应静态文件的请求。 (详见 :ref:`tutorial-static-files` )这个例子只能响应在 ``/path/to/your/static/files`` 目录下的文件请求，
因为 ``<filename>`` 这样的通配符定义不能匹配一个路径(路径中包含"/")。 为了响应子目录下的文件请求，我们需要更改 `path` 过滤器的定义。

::

  @route('/static/<filepath:path>')
  def server_static(filepath):
      return static_file(filepath, root='/path/to/your/static/files')

使用 ``root='./static/files'`` 这样的相对路径的时候，请注意当前工作目录 (``./``) 不一定是项目文件夹。

.. _tutorial-errorhandling:

错误页面
------------------------------------------------------------------------------

如果出错了，Bottle会显示一个默认的错误页面，提供足够的debug信息。你也可以使用 :func:`error` 函数来自定义你的错误页面::

  from bottle import error
  @error(404)
  def error404(error):
      return 'Nothing here, sorry'
      
从现在开始，在遇到404错误的时候，将会返回你在上面自定义的页面。传给error404函数的唯一参数，是一个 :exc:`HTTPError` 对象的实例。除此之外，这个回调函数与我们用来响应普通请求的回调函数没有任何不同。你可以从 :data:`request` 中读取数据， 往 :data:`response` 中写入数据和返回所有支持的数据类型，除了 :exc:`HTTPError` 的实例。

只有在你的应用返回或raise一个 :exc:`HTTPError` 异常的时候(就像 :func:`abort` 函数那样)，处理Error的函数才会被调用。更改 :attr:`Request.status` 或返回 :exc:`HTTPResponse` 不会触发错误处理函数。


.. _tutorial-output:

生成内容
==============================================================================

在纯WSGI环境里，你的应用能返回的内容类型相当有限。应用必须返回一个iterable的字节型字符串。你可以返回一个字符串(因为字符串是iterable的)，但这会导致服务器按字符来传输你的内容。Unicode字符串根本不允许。这不是很实用。

Bottle支持返回更多的内容类型，更具弹性。它甚至能在合适的情况下，在HTTP头中添加 `Content-Length` 字段和自动转换unicode编码。下面列出了所有你能返回的内容类型，以及框架处理方式的一个简述。

字典类型:
    上面已经提及，Python中的字典类型(或其子类)会被自动转换为JSON字符串。返回给浏览器的时候，HTTP头的 ``Content-Type`` 字段被自动设置为 `` application/json`` 。可十分简单地实现基于JSON的API。Bottle同时支持json之外的数据类型，详见 :ref:`tutorial-output-filter` 。

空字符串: ``False`` , ``None`` 或其它非真值:
    输出为空， ``Content-Length`` 设为0。

Unicode字符串:
    Unicode字符串 (or iterables yielding unicode strings) 被自动转码， ``Content-Type`` 被默认设置为utf8，接着视之为普通字符串(见下文)。

字节型字符串:
    Bottle将字符串当作一个整体来返回(而不是按字符来遍历)，并根据字符串长度添加 ``Content-Length`` 字段。包含字节型字符串的列表先被合并。其它iterable的字节型字符串不会被合并，因为它们也许太大来，耗内存。在这种情况下， ``Content-Length`` 字段不会被设置。
    
    (译者注：这段翻译不顺畅，保留原文)
    Bottle returns strings as a whole (instead of iterating over each char) and adds a ``Content-Length`` header based on the string length. Lists of byte strings are joined first. Other iterables yielding byte strings are not joined because they may grow too big to fit into memory. The ``Content-Length`` header is not set in this case.

:exc:`HTTPError` 或 :exc:`HTTPResponse` 的实例:
    返回它们和直接raise出来有一样的效果。对于 :exc:`HTTPError` 来说，会调用错误处理程序。详见 :ref:`tutorial-errorhandling` 。

文件对象:
    任何有 ``.read()`` 方法的对象都被当成一个file-like对象来对待，会被传给 WSGI Server 框架定义的 ``wsgi.file_wrapper`` callable对象来处理。一些WSGI Server实现会利用优化过的系统调用(sendfile)来更有效地传输文件，另外就是分块遍历。可选的HTTP头，例如 ``Content-Length`` 和 ``Content-Type`` 不会被自动设置。尽可能使用 :func:`send_file` 。详见 :ref:`tutorial-static-files` 。

Iterables and generators:
    你可以在回调函数中使用 ``yield`` 语句，或返回一个iterable的对象，只要该对象返回的是字节型字符串，unicode字符串， :exc:`HTTPError` 或 :exc:`HTTPResponse` 实例。不支持嵌套iterable对象，不好意思。注意，在iterable对象返回第一个非空值的时候，就会把HTTP状态码和HTTP头发送给浏览器。稍后再更改它们就起不到什么作用了。

以上列表的顺序非常重要。在你返回一个 :class:`str` 类的子类的时候，即使它有 ``.read()`` 方法，它依然会被当成一个字符串对待，而不是文件，因为字符串先被处理。


.. rubric:: 改变默认编码

Bottle使用 ``Content-Type`` 的 `charset` 参数来决定编码unicode字符串的方式。默认的 ``Content-Type`` 是 ``text/html;charset=UTF8`` ，可在 :attr:`Response.content_type` 属性中修改，或直接设置 :attr:`Response.charset` 的值。关于 :class:`Response` 对象的介绍，详见 :ref:`tutorial-response` 。

::

    from bottle import response
    @route('/iso')
    def get_iso():
        response.charset = 'ISO-8859-15'
        return u'This will be sent with ISO-8859-15 encoding.'

    @route('/latin9')
    def get_latin():
        response.content_type = 'text/html; charset=latin9'
        return u'ISO-8859-15 is also known as latin9.'

在极少情况下，Python中定义的编码名字和HTTP标准中的定义不一样。这样，你就必须同时修改 :attr:`Response.content_type`` (发送给客户端的)和设置 :attr:`Response.charset` 属性 (用于编码unicode)。


.. _tutorial-static-files:

静态文件
--------------------------------------------------------------------------------

你可直接返回文件对象。但我们更建议你使用 :func:`static_file` 来提供静态文件服务。它会自动猜测文件的mime-type，添加 ``Last-Modified`` 头，将文件路径限制在一个root文件夹下面来保证安全，且返回合适的HTTP状态码(由于权限不足导致的401错误，由于文件不存在导致的404错误)。它甚至支持 ``If-Modified-Since`` 头，如果文件未被修改，则直接返回 ``304 Not Modified`` 。你可指定MIME类型来避免其自动猜测。


::

    from bottle import static_file
    @route('/images/<filename:re:.*\.png>#')
    def send_image(filename):
        return static_file(filename, root='/path/to/image/files', mimetype='image/png')

    @route('/static/<filename:path>')
    def send_static(filename):
        return static_file(filename, root='/path/to/static/files')

如果确实需要，你可将 :func:`static_file` 的返回值当作异常raise出来。


.. rubric:: 强制下载

大多数浏览器在知道MIME类型的时候，会尝试直接调用相关程序来打开文件(例如PDF文件)。如果你不想这样，你可强制浏览器只是下载该文件，甚至提供文件名。::

    @route('/download/<filename:path>')
    def download(filename):
        return static_file(filename, root='/path/to/static/files', download=filename)

如果 ``download`` 参数的值为 ``True`` ，会使用原始的文件名。

.. _tutorial-error:

HTTP错误和重定向
--------------------------------------------------------------------------------

:func:`abort` 函数是生成HTTP错误页面的一个捷径。

::

    from bottle import route, abort
    @route('/restricted')
    def restricted():
        abort(401, "Sorry, access denied.")
        
为了将用户访问重定向到其他URL，你在 ``Location`` 中设置新的URL，接着返回一个 ``303 See Other`` 。 :func:`redirect` 函数可以帮你做这件事情。

::

    from bottle import redirect
    @route('/wrong/url')
    def wrong():
        redirect("/right/url")

你可以在第二个参数中提供另外的HTTP状态码。


.. note::
    这两个函数都会抛出 :exc:`HTTPError` 异常，终止回调函数的执行。

.. rubric:: 其他异常

除了 :exc:`HTTPResponse` 或 :exc:`HTTPError` 以外的其他异常，都会导致500错误，所以不会造成WSGI服务器崩溃。你将 ``bottle.app().catchall`` 的值设为 ``False`` 来关闭这种行为，以便在你的中间件中处理异常。


.. _tutorial-response:

:class:`Response` 对象
--------------------------------------------------------------------------------

诸如HTTP状态码，HTTP响应头，用户cookie等元数据都保存在一个名字为 :data:`response` 的对象里面，接着被传输给浏览器。你可直接操作这些元数据或使用一些更方便的函数。在API章节可查到所有相关API(详见 :class:`Response` )，这里主要介绍一些常用方法。


.. rubric:: 状态码

 `HTTP状态码 <http_code>`_ 控制着浏览器的行为，默认为 ``200 OK`` 。多数情况下，你不必手动修改 :attr:`Response.status` 的值，可使用 :func:`abort` 函数或return一个 :exc:`HTTPResponse` 实例(带有合适的状态码)。虽然所有整数都可当作状态码返回，但浏览器不知道如何处理 `HTTP标准 <http_code>`_ 中定义的那些状态码之外的数字，你也破坏了大家约定的标准。

.. rubric:: 响应头

``Cache-Control`` 和 ``Location`` 之类的响应头通过 :meth:`Response.set_header` 来定义。这个方法接受两个参数，一个是响应头的名字，一个是它的值，名字是大小写敏感的。

::

  @route('/wiki/<page>')
  def wiki(page):
      response.set_header('Content-Language', 'en')
      ...

大多数的响应头是唯一的，meaning that only one header per name is send to the client。一些特殊的响应头在一次response中允许出现多次。使用 :meth:`Response.add_header` 来添加一个额外的响应头，而不是 :meth:`Response.set_header` 。

::

    response.set_header('Set-Cookie', 'name=value')
    response.add_header('Set-Cookie', 'name2=value2')

请注意，这只是一个例子。如果你想使用cookie，详见 :ref:`ahead <tutorial-cookies>` 。


.. _tutorial-cookies:

Cookies
-------------------------------------------------------------------------------

Cookie是储存在浏览器配置文件里面的一小段文本。你可通过 :meth:`Request.get_cookie` 来访问已存在的Cookie，或通过 :meth:`Response.set_cookie` 来设置新的Cookie。

::

    @route('/hello')
    def hello_again():
        if request.get_cookie("visited"):
            return "Welcome back! Nice to see you again"
        else:
            response.set_cookie("visited", "yes")
            return "Hello there! Nice to meet you"
            
:meth:`Response.set_cookie` 方法接受一系列额外的参数，来控制Cookie的生命周期及行为。一些常用的设置如下:

* **max_age:**    最大有效时间，以秒为单位 (默认: ``None``)
* **expires:**    一个datetime对象或一个UNIX timestamp (默认: ``None``)
* **domain:**     可访问该Cookie的域名 (默认: 当前域名)
* **path:**       限制cookie的访问路径 (默认: ``/``)
* **secure:**     只允许在HTTPS链接中访问cookie (默认: off)
* **httponly:**   防止客户端的javascript读取cookie (默认: off, 要求python 2.6或以上版本)

如果 `expires` 和 `max_age` 两个值都没设置，cookie会在当前的浏览器session失效或浏览器窗口关闭后失效。在使用cookie的时候，应该注意一下几个陷阱。

* 在大多数浏览器中，cookie的最大容量为4KB。
* 一些用户将浏览器设置为不接受任何cookie。大多数搜索引擎也忽略cookie。确保你的应用在无cookie的时候也能工作。
* cookie被储存在客户端，也没被加密。你在cookie中储存的任何数据，用户都可以读取。更坏的情况下，cookie会被攻击者通过 `XSS <http://en.wikipedia.org/wiki/HTTP_cookie#Cookie_theft_and_session_hijacking>`_ 偷走，一些已知病毒也会读取浏览器的cookie。既然如此，就不要在cookie中储存任何敏感信息。
* cookie可以被伪造，不要信任cookie！

.. _tutorial-signed-cookies:

.. rubric:: Cookie签名

上面提到，cookie容易被客户端伪造。Bottle可通过加密cookie来防止此类攻击。你只需在读取和设置cookie的时候，通过 `secret` 参数来提供一个密钥。如果cookie未签名或密钥不匹配， :meth:`Request.get_cookie` 方法返回 ``None`` 

::

    @route('/login')
    def login():
        username = request.forms.get('username')
        password = request.forms.get('password')
        if check_user_credentials(username, password):
            response.set_cookie("account", username, secret='some-secret-key')
            return "Welcome %s! You are now logged in." % username
        else:
            return "Login failed."

    @route('/restricted')
    def restricted_area():
        username = request.get_cookie("account", secret='some-secret-key')
        if username:
            return "Hello %s. Welcome back." % username
        else:
            return "You are not logged in. Access denied."

例外，Bottle自动序列化储存在签名cookie里面的数据。你可在cookie中储存任何可序列化的对象(不仅仅是字符串)，只要对象大小不超过4KB。

.. warning:: 签名cookie在客户端不加密(译者注：即在客户端没有经过二次加密)，也没有写保护(客户端可使用之前的cookie)。给cookie签名的主要意义在于在cookie中存储序列化对象和防止伪造cookie，依然不要在cookie中存储敏感信息。










.. _tutorial-request:

请求数据 (Request Data)
==============================================================================

Bottle通过 ``request`` 对象来提供对HTTP相关元数据的访问，例如cookie，HTTP头，通过POST方法提交的表单。只要在回调函数中使用，该对象总是包含当前这次浏览器请求的信息。即使在多线程环境中，多个请求同时被响应，这依然有效。关于全局对象是如何做到线程安全的，详见 :doc:`contextlocal`.


.. note::

    Bottle在 :class:`FormsDict` 类的一个实例里(译者注：参见源码1763行左右)，存储大多数解析后的HTTP元数据。其行为和普通的字典类似，但有一些额外的特性: 所有在字典中的值都可像访问类的属性那样访问。这类虚拟的属性总是返回一个unicode字符串，如果缺失该属性，将返回一个空字符串。

    :class:`FormsDict` 类继承自 :class:`MultiDict` 类，一个key可存储多个值。标准的字典访问方法只会返回一个值，但可通过 :meth:`MultiDict.getall` 方法来获取一个包含所有值的list (或许为空)。
    
在API的章节，有关于API和特性的完整描述，这里我们只谈常见的用例和特性。



Cookies
--------------------------------------------------------------------------------

cookie存储在 :attr:`BaseRequest.cookie` 中，是  :class:`FormsDict` 类的一个实例。 :meth:`BaseRequest.get_cookie` 方法提供了对 :ref:`签名cookie <tutorial-signed-cookies>` 的访问。下面是一个基于cookie的计数器例子。

::


  from bottle import route, request, response
  @route('/counter')
  def counter():
      count = int( request.cookies.get('counter', '0') )
      count += 1
      response.set_cookie('counter', str(count))
      return 'You visited this page %d times' % count


HTTP头
--------------------------------------------------------------------------------

所有发送到客户端的HTTP头(例如 ``Referer``, ``Agent`` 和 ``Accept-Language``)存储在一个 :class:`WSGIHeaderDict` 中，可通过 :attr:`BaseRequest.headers` 访问。  :class:`WSGIHeaderDict` 是一个字典，其key大小写敏感。

::


  from bottle import route, request
  @route('/is_ajax')
  def is_ajax():
      if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
          return 'This is an AJAX request'
      else:
          return 'This is a normal request'


查询变量
--------------------------------------------------------------------------------

查询字符串(例如 ``/forum?id=1&page=5`` 中的 id=5和page=5)一般用于向服务器传输键值对。你可通过 :attr:`BaseRequest.query` ( :class:`FormsDict` 类的实例) 来访问，和通过 :attr:`BaseRequest.query_string` 来获取整个字符串。


::

  from bottle import route, request, response
  @route('/forum')
  def display_forum():
      forum_id = request.query.id
      page = request.query.page or '1'
      return 'Forum ID: %s (page %s)' % (forum_id, page)


POST表单数据和文件上传
-------------------------------

通过 ``POST`` 和 ``PUT`` 方式提交的请求也许会包含经过编码的表单数据。 :attr:`BaseRequest.forms` 字典包含了解析过后的文本字段， :attr:`BaseRequest.files` 字典存储上传的文件， :attr:`BaseRequest.POST` 同时包含两者。这三个都是 :class:`FormsDict` 类的实例，在你使用它们的时候才会被创建。上传的文件保存为一个包含其他元数据的 :class:`cgi.FieldStorage` 对象。最后，你可像访问文件一样，通过 :attr:`BaseRequest.body` 访问未经处理的请求内容。


下面是一个简单的文件上传例子。

.. code-block:: html

    <form action="/upload" method="post" enctype="multipart/form-data">
      <input type="text" name="name" />
      <input type="file" name="data" />
    </form>

::

    from bottle import route, request
    @route('/upload', method='POST')
    def do_upload():
        name = request.forms.name
        data = request.files.data
        if name and data and data.file:
            raw = data.file.read() # This is dangerous for big files
            filename = data.filename
            return "Hello %s! You uploaded %s (%d bytes)." % (name, filename, len(raw))
        return "You missed a field."


Unicode issues
-----------------------

在 **Python2** 中，所有的key和value都是byte-string。如果你需要unicode，可使用 :meth:`FormsDict.getunicode` 方法或像访问属性那样访问。这两种方法都试着将字符串转码(默认: utf8)，如果失败，将返回一个空字符串。无需捕获 :exc:`UnicodeError` 异常。

::

  >>> request.query['city']
  'G\xc3\xb6ttingen'  # A utf8 byte string
  >>> request.query.city
  u'Göttingen'        # The same string as unicode
  
在 **Python3** 中，所有的字符串都是unicode。但HTTP是基于字节的协议，在byte-string被传给应用之前，服务器必须将其转码。安全起见，WSGI协议建议使用ISO-8859-1 (即是latin1)，一个可反转的单字节编码，可被转换为其他编码。Bottle通过 :meth:`FormsDict.getunicode` 和属性访问实现了转码，但不支持字典形式的访问。通过字典形式的访问，将直接返回服务器返回的字符串，未经处理，这或许不是你想要的。


  >>> request.query['city']
  'GÃ¶ttingen' # An utf8 string provisionally decoded as ISO-8859-1 by the server
  >>> request.query.city
  'Göttingen'  # The same string correctly re-encoded as utf8 by bottle

如果你整个字典包含正确编码后的值(e.g. for WTForms)，可通过 :meth:`FormsDict.decode` 方法来获取一个转码后的拷贝(译者注：一个新的实例)。


WSGI 环境
--------------------------------------------------------------------------------

每一个  :class:`BaseRequest` 类的实例都包含一个WSGI环境的字典。最初存储在 :attr:`BaseRequest.environ` 中，但request对象也表现的像一个字典。大多数有用的数据都通过特定的方法或属性暴露了出来，但如果你想直接访问 `WSGI环境变量 <WSGI specification>`_ ，可以这样做。

::


  @route('/my_ip')
  def show_ip():
      ip = request.environ.get('REMOTE_ADDR')
      # or ip = request.get('REMOTE_ADDR')
      # or ip = request['REMOTE_ADDR']
      return "Your IP is: %s" % ip





.. _tutorial-templates:

模板
================================================================================

Bottle内置了一个快速的，强大的模板引擎，称为 :doc:`stpl` 。可通过 :func:`template` 函数或 :func:`view` 修饰器来渲染一个模板。只需提供模板的名字和传递给模板的变量。下面是一个简单的例子。

::


    @route('/hello')
    @route('/hello/<name>')
    def hello(name='World'):
        return template('hello_template', name=name)

这会加载 ``hello_template.tpl`` 模板文件，并提供 ``name`` 变量。默认情况，Bottle会在 ``./views/`` 目录查找模板文件(译者注：或当前目录)。可在 ``bottle.TEMPLATE_PATH`` 这个列表中添加更多的模板路径。

:func:`view` 修饰器允许你在回调函数中返回一个字典，并将其传递给模板，和 :func:`template` 函数做同样的事情。
 
 ::

    @route('/hello')
    @route('/hello/<name>')
    @view('hello_template')
    def hello(name='World'):
        return dict(name=name)

.. rubric:: 语法

.. highlight:: html+django

模板语法类似于Python的语法。它要确保语句块的正确缩进，所以你在写模板的时候无需担心会出现缩进问题。详细的语法描述可看 :doc:`stpl` 。


简单的模板例子

::

    %if name == 'World':
        <h1>Hello {{name}}!</h1>
        <p>This is a test.</p>
    %else:
        <h1>Hello {{name.title()}}!</h1>
        <p>How are you?</p>
    %end

.. rubric:: 缓存

模板在经过编译后被缓存在内存里。你在修改模板文件后，要调用 ``bottle.TEMPLATES.clear()`` 函数清除缓存才能看到效果。在debug模式下，缓存被禁用了，无需手动清除缓存。


.. highlight:: python




.. _plugins:

插件
================================================================================

.. versionadded:: 0.9

Bottle的核心功能覆盖了常见的使用情况，但是作为一个迷你框架，它有它的局限性。所以我们引入了插件机制，插件可以给框架添加其缺少的功能，集成第三方的库，或是自动化一些重复性的工作。

我们有一个不断增长的 :doc:`/plugins/index` 插件列表，大多数插件都被设计为可插拔的。有很大可能，你的问题已经被解决，而且已经有现成的插件可以使用了。如果没有现成的插件， :doc:`/plugindev` 有介绍如何开发一个插件。

插件扮演着各种各样的角色。例如， ``SQLitePlugin`` 插件给每个route的回调函数都添加了一个 ``db`` 参数，在回调函数被调用的时候，会新建一个数据库连接。这样，使用数据库就非常简单了。

::

    from bottle import route, install, template
    from bottle_sqlite import SQLitePlugin

    install(SQLitePlugin(dbfile='/tmp/test.db'))

    @route('/show/<post_id:int>')
    def show(db, post_id):
        c = db.execute('SELECT title, content FROM posts WHERE id = ?', (post_id,))
        row = c.fetchone()
        return template('show_post', title=row['title'], text=row['content'])

    @route('/contact')
    def contact_page():
        ''' 这个回调函数不需要数据库连接。因为没有"db"参数，sqlite插件会完全忽略这个回调函数。'''
        return template('contact')

其它插件或许在线程安全的 :data:`local` 对象里面发挥作用，改变 :data:`request` 对象的细节，过滤回调函数返回的数据或完全绕开回调函数。举个例子，一个用于登录验证的插件会在调用原先的回调函数响应请求之前，验证用户的合法性，如果是非法访问，则返回登录页面而不是调用回调函数。具体的做法要看插件是如何实现的。


整个应用的范围内安装插件
--------------------------------------------------------------------------------

可以在整个应用的范围内安装插件，也可以只是安装给某些route。大多数插件都可安全地安装给所有route，也足够智能，可忽略那些并不需要它们的route。

让我们拿 ``SQLitePlugin`` 插件举例，它只会影响到那些需要数据库连接的route，其它route都被忽略了。正因为如此，我们可以放心地在整个应用的范围内安装这个插件。

调用 :func:`install` 函数来安装一个插件

::

    from bottle_sqlite import SQLitePlugin
    install(SQLitePlugin(dbfile='/tmp/test.db'))

插件没有马上应用到所有route上面，它被延迟执行来确保没有遗漏任何route。你可以先安装插件，再添加route。有时，插件的安装顺序很重要，如果另外一个插件需要连接数据库，那么你就需要先安装操作数据库的插件。


.. rubric:: 卸载插件

调用 :func:`uninstall` 函数来卸载已经安装的插件

::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')
    install(sqlite_plugin)

    uninstall(sqlite_plugin) # 卸载指定插件
    uninstall(SQLitePlugin)  # 卸载这种类型的插件
    uninstall('sqlite')      # 卸载名字是sqlite的插件
    uninstall(True)          # 马上卸载所有插件
    
在任何时候，插件都可以被安装或卸载，即使是在服务器正在运行的时候。一些小技巧应用到了这个特征，例如在需要的时候安装一些供debug和性能测试的插件，但不可滥用这个特性。每一次安装或卸载插件的时候，route缓存都会被刷新，所有插件被重新加载。


.. note::
    模块层面的 :func:`install` 和 :func:`unistall` 函数会影响 :ref:`default-app` 。针对应用来管理插件，可使用 :class:`Bottle` 应用对象的相应方法。


安装给特定的route
--------------------------------------------------------------------------------

:func:`route` 修饰器的 ``apply`` 参数可以给指定的route安装插件

::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')

    @route('/create', apply=[sqlite_plugin])
    def create(db):
        db.execute('INSERT INTO ...')


插件黑名单
--------------------------------------------------------------------------------

如果你想显式地在一些route上面禁用某些插件，可使用 :func:`route` 修饰器的 ``skip`` 参数。

::

    sqlite_plugin = SQLitePlugin(dbfile='/tmp/test.db')
    install(sqlite_plugin)

    @route('/open/<db>', skip=[sqlite_plugin])
    def open_db(db):
        # 这次，插件不会修改'db'参数了
        if db in ('test', 'test2'):
            # The plugin handle can be used for runtime configuration, too.
            sqlite_plugin.dbfile = '/tmp/%s.db' % db
            return "Database File switched to: /tmp/%s.db" % db
        abort(404, "No such database.")

``skip`` 参数接受单一的值或是一个list。你可使用插件的名字，类，实例来指定你想要禁用的插件。如果 ``skip`` 的值为True，则禁用所有插件。

插件和子应用
--------------------------------------------------------------------------------

大多数插件只会影响到安装了它们的应用。因此，它们不应该影响通过 :meth:`Bottle.mount` 方法挂载上来的子应用。这里有一个例子。

::

    root = Bottle()
    root.mount('/blog', apps.blog)

    @root.route('/contact', template='contact')
    def contact():
        return {'email': 'contact@example.com'}

    root.install(plugins.WTForms())

在你挂载一个应用的时候，Bottle在主应用上面创建一个代理route，将所有请求转接给子应用。在代理route上，默认禁用了插件。如上所示，我们的 ``WTForms`` 插件影响了 ``/contact`` route，但不会影响挂载在root上面的 ``/blog`` 。

这个是一个合理的行为，但可被改写。下面的例子，在指定的代理route上面应用了插件。

::

    root.mount('/blog', apps.blog, skip=None)

这里存在一个小难题: 插件会整个子应用当作一个route看待，即是上面提及的代理route。如果想在子应用的每个route上面应用插件，你必须显式地在子应用上面安装插件。



开发
================================================================================

到目前为止，你已经学到一些开发的基础，并想写你自己的应用了吧？这里有一些小技巧可提高你的生产力。


.. _default-app:

默认应用
--------------------------------------------------------------------------------

Bottle维护一个全局的 :class:`Bottle` 实例的栈，模块层面的函数和修饰器使用栈顶实例作为默认应用。例如 :func:`route` 修饰器，相当于在默认应用上面调用了 :meth:`Bottle.route` 方法。

::

    @route('/')
    def hello():
        return 'Hello World'

对于小应用来说，这样非常方便，可节约你的工作量。但这同时意味着，在你的模块导入的时候，你定义的route就被安装到全局的默认应用中了。为了避免这种模块导入的副作用，Bottle提供了另外一种方法，显式地管理应用。

::

    app = Bottle()

    @app.route('/')
    def hello():
        return 'Hello World'

分离应用对象，大大提高了可重用性。其他开发者可安全地从你的应用中导入 ``app`` 对象，然后通过 :meth:`Bottle.mount` 方法来合并到其它应用中。

作为一种选择，你可通过应用栈来隔离你的route，依然使用方便的 ``route`` 修饰器。

::

    default_app.push()

    @route('/')
    def hello():
        return 'Hello World'

    app = default_app.pop()
    
:func:`app` 和 :func:`default_app` 都是 :class:`AppStack` 类的一个实例，实现了栈形式的API操作。你可push应用到栈里面，也可将栈里面的应用pop出来。在你需要导入一个第三方模块，但它不提供一个独立的应用对象的时候，尤其有用。

::

    default_app.push()

    import some.module

    app = default_app.pop()


.. _tutorial-debugging:


调试模式
--------------------------------------------------------------------------------

在开发的早期阶段，调试模式非常有用。

.. highlight:: python

::

    bottle.debug(True)

在调试模式下，当错误发生的时候，Bottle会提供更多的调试信息。同时禁用一些可能妨碍你的优化措施，检查你的错误设置。

下面是调试模式下会发生改变的东西，但这份列表不完整:

* 默认的错误页面会打印出运行栈。
* 模板不会被缓存。
* 插件被马上应用。

请确保不要在生产环境中使用调试模式。


自动加载
--------------------------------------------------------------------------------

在开发的时候，你需要不断地重启服务器来验证你最新的改动。自动加载功能可以替你做这件事情。在你编辑完一个模块文件后，它会自动重启服务器进程，加载最新版本的代码。

::

    from bottle import run
    run(reloader=True)

它的工作原理，主进程不会启动服务器，它使用相同的命令行参数，创建一个子进程来启动服务器。请注意，所有模块级别的代码都被执行了至少两次。

子进程中 ``os.environ['BOOTLE_CHILD']`` 变量的值被设为 ``True`` ，它运行一个不会自动加载的服务器。在代码改变后，主进程会终止掉子进程，并创建一个新的子进程。更改模板文件不会触发自动重载，请使用debug模式来禁用模板缓存。

自动加载需要终止子进程。如果你运行在Windows等不支持 ``signal.SIGINT`` (会在Python中raise ``KeyboardInterrupt`` 异常)的系统上，会使用 ``signal.SIGTERM`` 来杀掉子进程。在子进程被 ``SIGTERM`` 杀掉的时候，exit handlers和finally等语句不会被执行。


命令行接口
--------------------------------------------------------------------------------

.. versionadded: 0.10

从0.10版本开始，你可像一个命令行工具那样使用Bottle:

.. code-block:: console

    $ python -m bottle

    Usage: bottle.py [options] package.module:app

    选项:
      -h, --help            显示帮助信息并退出
      --version             显示版本号
      -b ADDRESS, --bind=ADDRESS
                            绑定socket到ADDRESS.
      -s SERVER, --server=SERVER
                            使用SERVER对应的后端服务器
      -p PLUGIN, --plugin=PLUGIN
                            安装插件
      --debug               启动到调试模式
      --reload              自动重载
      
`ADDRESS` 参数接受一个IP地址或IP:端口，其默认为 ``localhost:8080`` 。其它参数都很好地自我解释了。

插件和应用都通过一个导入表达式来指定。包含了导入的路径(例如: ``package.module`` )和模块命名空间内的一个表达式，两者用":"分开。下面是一个简单例子，详见 :func:`load` 。

.. code-block:: console

    # 从'myapp.controller'模块中获取'app'对象
    # 在80端口启动一个paste服务器，监听所有来源的请求
    python -m bottle -server paste -bind 0.0.0.0:80 myapp.controller:app

    # 启动一个自动重载的开发服务器，使用全局的默认应用。route定义在'test.py'文件中
    python -m bottle --debug --reload test

    # 安装一个自定义的调试插件，赋予一些参数
    python -m bottle --debug --reload --plugin 'utils:DebugPlugin(exc=True)'' test

    # 加载一个通过'myapp.controller.make_app()'按需创建的应用
    python -m bottle 'myapp.controller:make_app()''


部署
================================================================================

Bottle默认运行在内置的 `wsgiref <http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server>`_  服务器上面。这个单线程的HTTP服务器在开发的时候特别有用，但其性能低下，在服务器负载不断增加的时候也许会是性能瓶颈。

最早的解决办法是让Bottle使用 paste_ 或 cherrypy_ 等多线程的服务器。

::

    bottle.run(server='paste')
    
在 :doc:`deployment` 章节中，会介绍更多部署的选择。




.. _tutorial-glossary:

词汇表
========

.. glossary

::

   回调函数
   Programmer code that is to be called when some external action happens.
   In the context of web frameworks, the mapping between URL paths and
   application code is often achieved by specifying a callback function
   for each URL.

   修饰器
   A function returning another function, usually applied 
   as a function transformation using the ``@decorator`` syntax. 
   See `python documentation for function definition <http://docs.python.org/reference/compound_stmts.html#function>`_ for more about decorators.

   环境(environ)
   A structure where information about all documents under the root is
   saved, and used for cross-referencing.  The environment is pickled
   after the parsing stage, so that successive runs only need to read
   and parse new and changed documents.

   handler function
   A function to handle some specific event or situation. In a web
   framework, the application is developed by attaching a handler function
   as callback for each specific URL comprising the application.

   source directory
   The directory which, including its subdirectories, contains all
   source files for one Sphinx project.

