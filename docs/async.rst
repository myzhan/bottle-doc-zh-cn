异步应用入门
===================================

异步设计模式和 `WSGI <http://www.python.org/dev/peps/pep-3333/>`_ 的同步本质并不能很好地兼容。这就是为什么大部分的异步框架(tornado, twisted, ...)都实现了专有的API来暴露它们的异步特征。Bottle是一个WSGI框架，也继承了WSGI的同步本质，但是谢谢优秀的 `gevent项目 <http://www.gevent.org/>`_ ，我们可以使用Bottle来编写异步应用了。这份文档介绍了在Bottle如何使用异步WSGI。

同步WSGI的限制
-------------------------------

简单来说， `WSGI标准 (pep 3333) <http://www.python.org/dev/peps/pep-3333/>`_ 定义了下面这一个request/response的循环：每次请求到达的时候，应用中的callable会被调用一次，返回一个主体iterator。接着服务器会遍历该主体，分块写入socket。遍历完整个主体，就关闭客户端的连接。

足够简单，但是存在一个小问题：所有这些都是同步的。如果你的应用需要等待数据(IO, socket, 数据库, ...)，除了返回一个空字符串(忙等)，就只能阻塞当前进程了。两种办法都会占用当前进程，这样它就不能处理新的请求了。这样就导致每个线程只能处理一个请求了。

大部分服务器都限制了线程的数量，避免伴随它们而来的资源消耗。常见的是一个线程池，内有20个或更少数量的线程。一旦所有的线程都被占用了，任何新的请求都会阻塞。事实上，对于其他人来说，服务器已经宕机了。如果你想实现一个聊天程序，使用ajax轮询来获取实时消息，很快你就会受到线程数量的限制。这个程序也未免太小众了。 


救星，Greenlet
------------------------

大多数服务器的线程池都限制了线程池中线程的数量，避免创建和切换线程的代价。尽管和进程(fork)比起来，线程还是挺便宜的。但是也没便宜到可以接受为每一个请求创建一个线程。

`gevent <http://www.gevent.org/>`_ 模块添加了 *greenlet* 的支持。greenlet和传统的线程类似，但其创建只需消耗很少的资源。基于gevent的服务器可以生成成千上万的greenlet，为每个连接分配一个greenlet也毫无压力。阻塞greenlet，也不会影响到服务器接受新的请求。同时处理的连接数理论上是没有限制的。

这令创建异步应用难以置信的简单，因为它们看起来很想同步程序。基于gevent服务器实际上不是异步的，是大规模多线程。下面是一个例子。

::

    from gevent import monkey; monkey.patch_all()

    from time import sleep
    from bottle import route, run

    @route('/stream')
    def stream():
        yield 'START'
        sleep(3)
        yield 'MIDDLE'
        sleep(5)
        yield 'END'

    run(host='0.0.0.0', port=8080, server='gevent')

第一行很重要。它让gevent monkey-patch了大部分Python的阻塞API，让它们不阻塞当前经常，将CPU让给下一个greenlet。它实际上用基于gevent的伪线程替换了Python的线程。这就是你依然可以使用 ``time.sleep()`` 这个照常来说会阻塞线程的函数。如果这种monkey-patch的方式感令你感到不舒服，你依然可以使用gevent中相应的函数 ``gevent.sleep()`` 。

如果你运行了上面的代码，接着访问 ``http://localhost:8080/stream`` ，你可看到 `START`, `MIDDLE`, 和 `END` 这几个字样依次出现(用时大约8秒)。它像普通的线程一样工作，但是现在你的服务器能同时处理成千上万的连接了。


.. note::

    一些浏览器在开始渲染一个页面之前，会缓存确定容量的数据。在这些浏览器上，你需要返回更多的数据才能
    看到效果。另外，很多浏览器限制一个URL只使用一个连接。在这种情况下，你可以使用另外的浏览器，或
    性能测试工具(例如： `ab` 或 `httperf` )来测试性能。

事件回调函数
---------------

异步框架的常见设计模式(包括 tornado, twisted, node.js 和 friends)，是使用非阻塞的API，绑定回调函数到异步事件上面。在显式地关闭之前，socket会保持打开，以便稍后回调函数往socket里面写东西。下面是一个基于 `tornado <http://www.tornadoweb.org/documentation#non-blocking-asynchronous-requests>`_ 的例子。

::

    class MainHandler(tornado.web.RequestHandler):
        @tornado.web.asynchronous
        def get(self):
            worker = SomeAsyncWorker()
            worker.on_data(lambda chunk: self.write(chunk))
            worker.on_finish(lambda: self.finish())

主要的好处就是MainHandler能早早结束，在回调函数继续写socket来响应之前的请求的时候，当前线程能继续接受新的请求。这样就是为什么这类框架能同时处理很多请求，只使用很少的操作系统线程。

对于Gevent和WSGI，情况就不一样了：首先，早早结束没有好处，因为我们的(伪)线程池已经没有限制了。第二，我们不能早早结束，因为这样会关闭socket(WSGI要求如此)。第三，我们必须返回一个iterator，以遵守WSGI的约定。

为了遵循WSGI规范，我们只需返回一个iterable的实体，异步地将其写回客户端。在 `gevent.queue <http://www.gevent.org/gevent.queue.html>`_ 的帮助下，我们可以 *模拟* 一个脱管的socket，上面的例子可写成这样。

::

    @route('/fetch')
    def fetch():
        body = gevent.queue.Queue()
        worker = SomeAsyncWorker()
        worker.on_data(lambda chunk: body.put(chunk))
        worker.on_finish(lambda: body.put(StopIteration))
        return body

从服务器的角度来看，queue对象是iterable的。如果为空，则阻塞，一旦遇到 ``StopIteration`` 则停止。这符合WSGI规范。从应用的角度来看，queue对象表现的像一个不会阻塞socket。你可以在任何时刻写入数据，pass it around，甚至启动一个新的(伪)线程，异步写入。这是在大部分情况下，实现长轮询。


最后： WebSockets
-------------------

让我们暂时忘记底层的细节，来谈谈WebSocket。既然你正在阅读这篇文章，你有可能已经知道什么是WebSocket了，一个在浏览器(客户端)和Web应用(服务端)的双向的交流通道。

感谢 `gevent-websocket <http://pypi.python.org/pypi/gevent-websocket/>`_ 包帮我们做的工作。下面是一个WebSocket的简单例子，接受消息然后将其发回客户端。

::

    from bottle import request, Bottle, abort
    app = Bottle()

    @app.route('/websocket')
    def handle_websocket():
        wsock = request.environ.get('wsgi.websocket')
        if not wsock:
            abort(400, 'Expected WebSocket request.')

        while True:
            try:
                message = wsock.receive()
                wsock.send("Your message was: %r" % message)
            except WebSocketError:
                break

    from gevent.pywsgi import WSGIServer
    from geventwebsocket import WebSocketHandler, WebSocketError
    server = WSGIServer(("0.0.0.0", 8080), app,
                        handler_class=WebSocketHandler)
    server.serve_forever()

while循环直到客户端关闭连接的时候才会终止。You get the idea :)

客户端的JavaScript API也十分简洁明了。

::

    <!DOCTYPE html>
    <html>
    <head>
      <script type="text/javascript">
        var ws = new WebSocket("ws://example.com:8080/websocket");
        ws.onopen = function() {
            ws.send("Hello, world");
        };
        ws.onmessage = function (evt) {
            alert(evt.data);
        };
      </script>
    </head>
    </html>

