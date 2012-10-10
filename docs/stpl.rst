===========================
SimpleTemplate 模板引擎
===========================

.. currentmodule:: bottle

Bottle自带了一个快速，强大，易用的模板引擎，名为 *SimpleTemplate* 或简称为 *stpl* 。它是 :func:`view` 和 :func:`template` 两个函数默认调用的模板引擎。接下来会介绍该引擎的模板语法和一些常见用例。


.. rubric:: 基础API :

:class:`SimpleTemplate` 类实现了 :class:`BaseTemplate` 接口

::

   >>> from bottle import SimpleTemplate
   >>> tpl = SimpleTemplate('Hello {{name}}!')
   >>> tpl.render(name='World')
   u'Hello World!'

简单起见，我们在例子中使用 :func:`template` 函数

::

   >>> from bottle import template
   >>> template('Hello {{name}}!', name='World')
   u'Hello World!'

注意，编译模板和渲染模板是两件事情，尽管 :func:`template` 函数隐藏了这一事实。通常，模板只会被编译一次，然后会被缓存起来，但是会根据不同的参数，被多次渲染。

:class:`SimpleTemplate` 的语法
==============================

虽然Python是一门强大的语言，但它对空白敏感的语法令其很难作为一个模板语言。SimpleTemplate移除了一些限制，允许你写出干净的，有可读性的，可维护的模板，且保留了Python的强大功能。

.. warning::

 :class:`SimpleTemplate` 模板会被编译为Python字节码，且在每次通过 :meth:`SimpleTemplate.render` 渲染的时候执行。请不要渲染不可靠的模板！它们也许包含恶意代码。

内嵌语句
-----------------

你已经在上面的"Hello World!"例子中学习到了 ``{{...}}`` 语句的用法。只要在 ``{{...}}`` 中的Python语句返回一个字符串或有一个字符串的表达形式，它就是一个有效的语句。

::

  >>> template('Hello {{name}}!', name='World')
  u'Hello World!'
  >>> template('Hello {{name.title() if name else "stranger"}}!', name=None)
  u'Hello stranger!'
  >>> template('Hello {{name.title() if name else "stranger"}}!', name='mArC')
  u'Hello Marc!'

{{}}中的Python语句会在渲染的时候被执行，可访问传递给 :meth:`SimpleTemplate.render` 方法的所有参数。默认情况下，自动转义HTML标签以防止 `XSS <http://en.wikipedia.org/wiki/Cross-Site_Scripting>`_ 攻击。可在语句前加上"!"来关闭自动转义。

::

  >>> template('Hello {{name}}!', name='<b>World</b>')
  u'Hello &lt;b&gt;World&lt;/b&gt;!'
  >>> template('Hello {{!name}}!', name='<b>World</b>')
  u'Hello <b>World</b>!'

.. highlight:: html+django

嵌入Pyhton代码
--------------------

一行以 ``%`` 开头，表明这一行是Python代码。它和真正的Python代码唯一的区别，在于你需要显式地在末尾添加 ``%end`` 语句，表明一个代码块结束。这样你就不必担心Python代码中的缩进问题， *SimpleTemplate* 模板引擎的parser帮你处理了。不以 ``%`` 开头的行，被当作普通文本来渲染。

::

  %if name:
    Hi <b>{{name}}</b>
  %else:
    <i>Hello stranger</i>
  %end

只有在行首的 ``%`` 字符才有意义，可以使用 ``%%`` 来转义。

::

  这一行包含 % 但不是Python代码。
  %% 以'%'开头的一行
  %%% 以'%%'开头的一行

防止换行
-----------------------

你可以在一行代码前面加上 ``\\`` 来防止换行。

::

  <span>\\
  %if True:
  nobreak\\
  %end
  </span>

该模板的输出::

  <span>nobreak</span>

``%include`` 语句
--------------------------

你可以使用 ``%include sub_template [kwargs]`` 语句来包含其他模板。 ``sub_template`` 参数是模板的文件名或路径。 ``[kwargs]`` 部分是以逗号分开的键值对，是传给其他模板的参数。 ``**kwargs`` 这样的语法来传递一个字典也是允许的。

::

  %include header_template title='Hello World'
  <p>Hello World</p>
  %include footer_template

``%rebase`` 语句
-------------------------

``%rebase base_template [kwargs]`` 语句会渲染 ``base_template`` 这个模板，而不是原先的模板。然后base_template中使用一个空 ``%include`` 语句来包含原先的模板，并可访问所有通过 ``kwargs`` 传过来的参数。这样就可以使用模板来封装另一个模板，或者是模拟某些模板引擎中的继承机制。

让我们假设，你现在有一个与内容有关的模板，想在它上面加上一层普通的HTML层。为了避免include一堆模板，你可以使用一个基础模板。

名为 ``layout.tpl`` 的基础模板

::

  <html>
  <head>
    <title>{{title or 'No title'}}</title>
  </head>
  <body>
    %include
  </body>
  </html>

名为 ``content.tpl`` 的主模板

::

  This is the page content: {{content}}
  %rebase layout title='Content Title'

渲染 ``content.tpl``

.. code-block:: python

  >>> print template('content', content='Hello World!')

.. code-block:: html

  <html>
  <head>
    <title>Content Title</title>
  </head>
  <body>
    This is the page content: Hello World!
  </body>
  </html>

一个更复杂的使用场景involves chained rebases and multiple content blocks. ``block_content.tpl`` 模板定义了两个函数，然后将它们传给 ``columns.tpl`` 这个基础模板。

::

  %def leftblock():
    Left block content.
  %end
  %def rightblock():
    Right block content.
  %end
  %rebase columns leftblock=leftblock, rightblock=rightblock, title=title

``columns.tpl`` 这个基础模板使用两个callable(译者注：Python中除了函数是callable的，类也可以是callable的，在这个例子里，是函数)来渲染分别位于左边和右边的两列。然后将其自身封装在之前定义的 ``layout.tpl`` 模板里面。

::

  %rebase layout title=title
  <div style="width: 50%; float:left">
    %leftblock()
  </div>
  <div style="width: 50%; float:right">
    %rightblock()
  </div>

让我们看一看 ``block_content.tpl`` 模板的输出

.. code-block:: python

  >>> print template('block_content', title='Hello World!')

.. code-block:: html

  <html>
  <head>
    <title>Hello World</title>
  </head>
  <body>
  <div style="width: 50%; float:left">
    Left block content.
  </div>
  <div style="width: 50%; float:right">
    Right block content.
  </div>
  </body>
  </html>

模板内置函数 (Namespace Functions)
------------------------------------

在模板中访问一个未定义的变量会导致 :exc:`NameError` 异常，并立即终止模板的渲染。
这是Python的正常行为，并不奇怪。在抛出异常之前，你无法检查变量是否被定义。这在你想让输入更灵活，
或想在不同情况下使用同一个模板的时候，就很烦人了。SimpleTemplate模板引擎内置了三个函数来帮你
解决这个问题，可以在模板的任何地方使用它们。

.. currentmodule:: stpl

.. function:: defined(name)

    如果变量已定义则返回True，反之返回False。

.. function:: get(name, default=None)

    返回该变量，或一个默认值

.. function:: setdefault(name, default)

    如果该变量未定义，则定义它，赋一个默认值，返回该变量

下面是使用了这三个函数的例子，实现了模板中的可选参数。

::

    % setdefault('text', 'No Text')
    <h1>{{get('title', 'No Title')}}</h1>
    <p> {{ text }} </p>
    % if defined('author'):
      <p>By {{ author }}</p>
    % end

.. currentmodule:: bottle


:class:`SimpleTemplate` API
==============================

.. autoclass:: SimpleTemplate
   :members:

已知Bug
==============================

不兼容某些Python语法,例子如下:

  * 多行语句语句必须以 ``\`` 结束, 如果出现了注释, 则不能再包含其他 ``#`` 字符.
  * 不支持多行字符串(译者注:"""  """这样的字符串)
