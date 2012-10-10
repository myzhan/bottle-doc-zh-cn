.. module:: bottle


========================
插件开发指南
========================

这份指南介绍了插件的API，以及如何编写自己的插件。我建议先阅读 :ref:`plugins` 这一部分，再看这份指南。 :doc:`/plugins/index` 这里也有一些实际的例子。

.. note::

    这是一份初稿。如果你发现了任何错误，或某些部分解释的不够清楚，请通过 `邮件列表 <mailto:bottlepy@googlegroups.com>`_ 或  `bug report <https://github.com/defnull/bottle/issues>`_ 告知。


插件工作方式：基础知识
============================

插件的API是通过Python的 `修饰器 <http://docs.python.org/glossary.html#term-decorator>`_ 来实现的。简单来说，一个插件就是应用在route回调函数上的修饰器。

当然，例子被我们简化了，除了作为route的回调函数修饰器，插件还可以做更多的事情，先看看代码吧。

::

    from bottle import response, install
    import time

    def stopwatch(callback):
        def wrapper(*args, **kwargs):
            start = time.time()
            body = callback(*args, **kwargs)
            end = time.time()
            response.headers['X-Exec-Time'] = str(end - start)
            return body
        return wrapper

    bottle.install(stopwatch)

这个插件计算每次请求的响应时间，并在响应头中添加了 ``X-Exec-Time`` 字段。如你所见，插件返回了一个wrapper函数，由它来调用原先的回调函数。这就是修饰器的常见工作方式了。

最后一行，将该插件安装到Bottle的默认应用里面。这样，应用中的所有route都会应用这个插件了。就是说，每次请求都会调用 ``stopwatch()`` ，更改了route的默认行为。

插件是按需加载的，就是在route第一次被访问的时候加载。为了在多线程环境下工作，插件应该是线程安全的。在大多数情况下，这都不是一个问题，但务必提高警惕。

一旦route中使用了插件后，插件中的回调函数会被缓存起来，接下来都是直接使用缓存中的版本来响应请求。意味着每个route只会请求一次插件。在应用的插件列表变化的时候，这个缓存会被清空。你的插件应当可以多次修饰同一个route。

这种修饰器般的API受到种种限制。你不知道route或相应的应用对象是如何被修饰的，也不知道如何有效地存储那些在route之间共享的数据。但别怕！插件不仅仅是修饰器函数。只要一个插件是callable的或实现了一个扩展的API(后面会讲到)，Bottle都可接受。扩展的API给你更多的控制权。


插件 API
==========

``Plugin`` 类不是一个真正的类(你不能从bottle中导入它)，它只是一个插件需要实现的接口。只要一个对象实现了以下接口，Bottle就认可它作为一个插件。

.. class:: Plugin(object)

    插件应该是callable的，或实现了 :meth:`apply` 方法。如果定义了 :meth:`apply` 方法，那么会优先调用，而不是直接调用插件。其它的方法和属性都是可选的。

    .. attribute :: name

        :meth:`Bottle.uninstall` 方法和 :meth:`Bottle.route()` 中的 `skip` 参数都接受一个与名字有关的字符串，对应插件或其类型。只有插件中有一个name属性的时候，这才会起作用。

    .. attribute :: api

        插件的API还在逐步改进。这个整形数告诉Bottle使用哪个版本的插件。如果没有这个属性，Bottle默认使用第一个版本。当前版本是 ``2`` 。详见 :ref:`plugin-changelog` 。

    .. method :: setup(self, app)

        插件被安装的时候调用(见 :meth:`Bottle.install` )。唯一的参数是相应的应用对象。

    .. method :: __call__(self, callback)

        如果没有定义 :meth:`apply` 方法，插件本身会被直接当成一个修饰器使用(译者注：Python的Magic Method，调用一个类即是调用类的__call__函数)，应用到各个route。唯一的参数就是其所修饰的函数。这个方法返回的东西会直接替换掉原先的回调函数。如果无需如此，则直接返回未修改过的回调函数即可。

    .. method :: apply(self, callback, route)

        如果存在，会优先调用，而不调用 :meth:`__call__` 。额外的 `route` 参数是 :class:`Route` 类的一个实例，提供很多该route信息和上下文。详见 :ref:`route-context` 。

    .. method :: close(self)

        插件被卸载或应用关闭的时候被调用，详见 :meth:`Bottle.uninstall` 或 :meth:`Bottle.close` 。

:meth:`Plugin.setup` 方法和 :meth:`Plugin.close` 方法 *不* 会被调用，如果插件是通过 :meth:`Bottle.route` 方法来应用到route上面的，但会在安装插件的时候被调用。


.. _plugin-changelog:

插件API的改动
------------------

插件的API还在不断改进中。在Bottle 0.10版本中的改动，定位了route上下文字典中已确定的问题。为了保持对0.9版本插件的兼容，我们添加了一个可选的 :attr:`Plugin.api` 属性，告诉Bottle使用哪个版本的API。API之间的不同点总结如下。

* **Bottle 0.9 API 1** (无 :attr:`Plugin.api` 属性)

  * Original Plugin API as described in the 0.9 docs.

* **Bottle 0.10 API 2** ( :attr:`Plugin.api` 属性为2)

  * :meth:`Plugin.apply` 方法中的 `context` 参数，现在是 :class:`Route` 类的一个实例，不再是一个上下文字典。

.. _route-context:


Route上下文
=================

:class:`Route` 的实例被传递给 :meth:`Plugin.apply` 函数，以提供更多该route的相关信息。最重要的属性总结如下。

===========  =================================================================
属性           描述
===========  =================================================================
app          安装该route的应用对象
rule         route规则的字符串 (例如： ``/wiki/:page``)
method       HTTP方法的字符串(例如： ``GET``)
callback     未应用任何插件的原始回调函数，用于内省。
name         route的名字，如未指定则为 ``None``
plugins      route安装的插件列表，除了整个应用范围内的插件，额外添加的(见 :meth:`Bottle.route` )
skiplist     应用安装了，但该route没安装的插件列表(见 meth:`Bottle.route` )
config       传递给 :meth:`Bottle.route` 修饰器的额外参数，存在一个字典中，用于特定的设置和元数据
===========  =================================================================

对你的应用而言， :attr:`Route.config` 也许是最重要的属性了。记住，这个字典会在所有插件中共享，建议添加一个独一无二的前缀。如果你的插件需要很多设置，将其保存在 `config` 字典的一个独立的命名空间吧。防止插件之间的命名冲突。


改变 :class:`Route` 对象
----------------------------------

:class:`Route` 的一些属性是不可变的，改动也许会影响到其它插件。坏主意就是，monkey-patch一个损坏的route，而不是提供有效的帮助信息来让用户修复问题。

在极少情况下，破坏规则也许是恰当的。在你更改了 :class:`Route` 实例后，抛一个 :exc:`RouteReset` 异常。这会从缓存中删除当前的route，并重新应用所有插件。无论如何，router没有被更新。改变 `rule` 或 `method` 的值并不会影响到router，只会影响到插件。这个情况在将来也许会改变。


运行时优化
=====================

插件应用到route以后，被插件封装起来的回调函数会被缓存，以加速后续的访问。如果你的插件的行为依赖一些设置，你需要在运行时更改这些设置，你需要在每次请求的时候读取设置信息。够简单了吧。

然而，为了性能考虑，也许值得根据当前需求，选择一个不同的封装，通过闭包，或在运行时使用、禁用一个插件。让我们拿内置的HooksPlugin作为一个例子(译者注：可在bottle.py搜索该实现)：如果没有安装任何钩子，这个插件会从所有受影响的route中删除自身，不做任何工作。一旦你安装了第一个钩子，这个插件就会激活自身，再次工作。

为了达到这个目的，你需要控制回调函数的缓存： :meth:`Route.reset` 函数清空单一route的缓存， :meth:`Bottle.reset` 函数清空所有route的缓存。在下一次请求的时候，所有插件被重新应用到route上面，就像第一次请求时那样。

如果在route的回调函数里面调用，两种方法都不会影响当前的请求。当然，可以抛出一个 :exc:`RouteReset` 异常，来改变当前的请求。


插件例子： SQLitePlugin
============================

这个插件提供对sqlite3数据库的访问，如果route的回调函数提供了关键字参数(默认是"db")，则"db"可做为数据库连接，如果route的回调函数没有提供该参数，则忽略该route。wrapper不会影响返回值，但是会处理插件相关的异常。 :meth:`Plugin.setup` 方法用于检查应用，查找冲突的插件。

::

    import sqlite3
    import inspect

    class SQLitePlugin(object):
        ''' This plugin passes an sqlite3 database handle to route callbacks
        that accept a `db` keyword argument. If a callback does not expect
        such a parameter, no connection is made. You can override the database
        settings on a per-route basis. '''

        name = 'sqlite'
        api = 2

        def __init__(self, dbfile=':memory:', autocommit=True, dictrows=True,
                     keyword='db'):
             self.dbfile = dbfile
             self.autocommit = autocommit
             self.dictrows = dictrows
             self.keyword = keyword

        def setup(self, app):
            ''' Make sure that other installed plugins don't affect the same
                keyword argument.'''
            for other in app.plugins:
                if not isinstance(other, SQLitePlugin): continue
                if other.keyword == self.keyword:
                    raise PluginError("Found another sqlite plugin with "\
                    "conflicting settings (non-unique keyword).")

        def apply(self, callback, context):
            # Override global configuration with route-specific values.
            conf = context.config.get('sqlite') or {}
            dbfile = conf.get('dbfile', self.dbfile)
            autocommit = conf.get('autocommit', self.autocommit)
            dictrows = conf.get('dictrows', self.dictrows)
            keyword = conf.get('keyword', self.keyword)

            # Test if the original callback accepts a 'db' keyword.
            # Ignore it if it does not need a database handle.
            args = inspect.getargspec(context.callback)[0]
            if keyword not in args:
                return callback

            def wrapper(*args, **kwargs):
                # Connect to the database
                db = sqlite3.connect(dbfile)
                # This enables column access by name: row['column_name']
                if dictrows: db.row_factory = sqlite3.Row
                # Add the connection handle as a keyword argument.
                kwargs[keyword] = db

                try:
                    rv = callback(*args, **kwargs)
                    if autocommit: db.commit()
                except sqlite3.IntegrityError, e:
                    db.rollback()
                    raise HTTPError(500, "Database Error", e)
                finally:
                    db.close()
                return rv

            # Replace the route callback with the wrapped one.
            return wrapper

这个插件十分有用，已经和Bottle提供的那个版本很类似了(译者注：=。= 一模一样)。只要60行代码，还不赖嘛！下面是一个使用例子。

::

    sqlite = SQLitePlugin(dbfile='/tmp/test.db')
    bottle.install(sqlite)

    @route('/show/:page')
    def show(page, db):
        row = db.execute('SELECT * from pages where name=?', page).fetchone()
        if row:
            return template('showpage', page=row)
        return HTTPError(404, "Page not found")

    @route('/static/:fname#.*#')
    def static(fname):
        return static_file(fname, root='/some/path')

    @route('/admin/set/:db#[a-zA-Z]+#', skip=[sqlite])
    def change_dbfile(db):
        sqlite.dbfile = '/tmp/%s.db' % db
        return "Switched DB to %s.db" % db

第一个route提供了一个"db"参数，告诉插件它需要一个数据库连接。第二个route不需要一个数据库连接，所以会被插件忽略。第三个route确实有一个"db"参数，但显式的禁用了sqlite插件，这样，"db"参数不会被插件修改，还是包含URL传过来的那个值。
