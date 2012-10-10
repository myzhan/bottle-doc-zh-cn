================================================================================
URL映射
================================================================================

Bottle内置一个强大的route引擎，可以给每个浏览器请求找到正确的回调函数。 :ref:`tutorial <tutorial-routing>` 中已经介绍了一些基础知识。接下来是一些进阶知识和route的规则。

route的语法
--------------------------------------------------------------------------------

:class:`Router` 类中明确区分两种类型的route： **静态route** (例如 ``/contact`` )和 **动态route** (例如 ``/hello/<name>`` )。包含了 *通配符* 的route即是动态route，除此之外的都是静态的。

.. versionchanged:: 0.10

包含通配符，最简单的形式就是将其放到一对<>里面(例如 ``<name>`` )。在同一个route里面，这个变量名需要是唯一的。因为稍后会将其当作参数传给回调函数，所以这个变量名的第一个字符应该是字母。

每一个通配符匹配一个或多个字符，直到遇到 ``/`` 。类似于 ``[^/]+`` 这样一个正则表达式，确保在route包含多个通配符的时候不出现歧义。


``/<action>/<item>`` 这个规则匹配的情况如下

:

============ =========================================
Path         Result
============ =========================================
/save/123    ``{'action': 'save', 'item': '123'}``
/save/123/   `不匹配`
/save/       `不匹配`
//123        `不匹配`
============ =========================================

你可通过过滤器来改变这一行为，稍后会介绍。

通配符过滤器
--------------------------------------------------------------------------------

.. versionadded:: 0.10

过滤器被用于定义更特殊的通配符，可在URL中"被匹配到的部分"被传递给回调函数之前，处理其内容。可通过 ``<name:filter>`` 或 ``<name:filer:config>`` 这样的语句来声明一个过滤器。"config"部分的语法由被使用的过滤器决定。

Bottle中已实现以下过滤器：

* **:int** 匹配一个整形数，并将其转换为int
* **:float** 同上，匹配一个浮点数
* **:path** 匹配所有字符，包括'/'
* **:re[:config]** 允许在config中写一个正则表达式

你可在route中添加自己写的过滤器。过滤器是一个有三个返回值的函数：一个正则表达式，一个callable的对象(转换URL片段为Python对象)，另一个callable对象(转换Python对象为URL片段)。过滤器仅接受一个参数，就是设置字符串(译者注：例如re过滤器的config部分)。以下是一个过滤器的例子，将逗号分开的字符串转换为数字列表。

::

    app = Bottle()

    def list_filter(config):
        ''' 匹配这样的字符串:'1,2,3,4,5,6' '''
        delimiter = config or ','
        regexp = r'\d+(%s\d)*' % re.escape(delimiter)

        def to_python(match):
            return map(int, match.split(delimiter))
        
        def to_url(numbers):
            return delimiter.join(map(str, numbers))
        
        return regexp, to_python, to_url

    app.router.add_filter('list', list_filter)

    @app.route('/follow/<ids:list>')
    def follow_users(ids):
        for id in ids:
            ...


旧语法
--------------------------------------------------------------------------------

.. versionchanged:: 0.10

在 **Bottle 0.10** 版本中引入了新的语法，来简单化一些常见用例，但依然兼容旧的语法。新旧语法的区别如下。

=================== ====================
旧语法                新语法
=================== ====================
``:name``           ``<name>``
``:name#regexp#``   ``<name:re:regexp>``
``:#regexp#``       ``<:re:regexp>``
``:##``             ``<:re>``
=================== ====================

请尽量在新项目中避免使用旧的语法，虽然它现在还没被废弃，但终究会的。


route的顺序 (URL映射的顺序)
--------------------------------------------------------------------------------

在通配符和正则表达式的支持下，我们是可以定义重叠的route的。如果多个route匹配同一个URL，它们的行为也许会变得难以琢磨。为了了解这里面究竟发生了什么事情，你需要了解route被执行的顺序。(译者注：router会检查route的顺序)

首先，route是按照它们的规则来分组的。两个规则相同但回调函数不同的route分到同一组，第一个route决定这两个route的位置。如果两个route完全相同(同样的规则，同样的回调函数)，则先定义的route会被后来的route替换掉，但它的位置位置会被保留下来。

(译者注：我搞不明白这里发生了什么，保留原文)
First you should know that routes are grouped by their path rule. Two routes with the same path rule but different methods are grouped together and the first route determines the position of both routes. Fully identical routes (same path rule and method) replace previously defined routes, but keep the position of their predecessor.

基于性能考虑，默认先查找静态route，这个默认设置可以被改掉。如果没有静态route匹配浏览器请求，则会按route被定义的顺序，去查找动态route，只要找到一个匹配的动态route，便不会继续查找。如果也找不到匹配请求的动态route，则返回一个404错误。

第二步，会检查浏览器请求的HTTP方法。如果不能精确匹配，且浏览器发的是HEAD请求，那个router会按照GET方法去查找相应的route，否则便按照ANY方法去查找route。如果都失败了，则返回一个405错误。

下面这个例子也许会令你迷惑

::

    @route('/<action>/<name>', method='GET')
    @route('/save/<name>', method='POST')

第二个route永远不会被命中，因为第一个route已经能匹配该URL，所以router会在查找到第一个route后停止查找，接着按照浏览器请求的HTTP方法来查找，如果找不到则返回405错误。

看起来挺复杂的，确实很复杂。这是为了性能而付出的代价。最好是避免创建容易造成歧义的route。将来或许会改变route实现的细节，我们在慢慢改善中。


显式的route配置
--------------------------------------------------------------------------------

route修饰器也可以直接当作函数来调用。在复杂的部署中，这种方法或许更灵活，直接由你来控制“何时”及“如何”配置route。

下面是一个简单的例子

::

    def setup_routing():
        bottle.route('/', method='GET', index)
        bottle.route('/edit', method=['GET', 'POST'], edit)

实际上，bottle可以是任何 :class:`Bottle` 类的实例

::

    def setup_routing(app):
        app.route('/new', method=['GET', 'POST'], form_new)
        app.route('/edit', method=['GET', 'POST'], form_edit)

    app = Bottle()
    setup_routing(app)

