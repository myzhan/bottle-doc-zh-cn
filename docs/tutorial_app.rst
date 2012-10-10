.. _Bottle: http://bottle.paws.org
.. _Python: http://www.python.org
.. _SQLite: http://www.sqlite.org
.. _Windows: http://www.sqlite.org/download.html
.. _PySQLite: http://pypi.python.org/pypi/pysqlite/
.. _`decorator statement`: http://docs.python.org/glossary.html#term-decorator
.. _`Python DB API`: http://www.python.org/dev/peps/pep-0249/
.. _`WSGI reference Server`: http://docs.python.org/library/wsgiref.html#module-wsgiref.simple_server
.. _Cherrypy: http://www.cherrypy.org/
.. _Fapws3: http://github.com/william-os4y/fapws3
.. _Flup: http://trac.saddi.com/flup
.. _Paste: http://pythonpaste.org/
.. _Apache: http://www.apache.org
.. _`Bottle documentation`: http://bottlepy.org/docs/dev/tutorial.html
.. _`mod_wsgi`: http://code.google.com/p/modwsgi/
.. _`json`: http://www.json.org

===============================
Tutorial: Todo-List 应用
===============================

.. note::

    这份教程是 `noisefloor <http://github.com/noisefloor>`_ 编写的，并在不断完善中。

这份教程简单介绍了Bottle框架，目的是让你看完后能在项目中使用Bottle。它没有涵盖所有东西，但介绍了URL映射，模板，处理GET/POST请求等基础知识。

读懂这份教程，你不需要事先了解WSGI标准，Bottle也一直避免用户直接接触WSGI标准。但你需要了解 Python_ 这门语言。更进一步，这份教程中的例子需要从SQL数据库中读写数据，所以事先了解一点SQL知识是很有帮助的。例子中使用了 SQLite_ 来保存数据。因为是网页应用，所以事先了解一点HTML的知识也很有帮助。

作为一份教程，我们的代码尽可能做到了简明扼要。尽管教程中的代码能够工作，但是我们还是不建议你在公共服务器中使用教程中的代码。如果你想要这样做，你应该添加足够的错误处理，并且加密你的数据库，处理用户的输入。


.. contents:: 目录

目标
===========

在这份教程结束的时候，我们将完成一个简单的，基于Web的ToDo list(待办事项列表)。列表中的每一个待办事项都包含一条文本(最长100个字符)和一个状态(0表示关闭，1表示开启)。通过网页，已开启的待办事项可以被查看和编辑，可添加待办事项到列表中。

在开发过程中，所有的页面都只可以通过 ``localhost`` 来访问，完了会介绍如何将应用部署到"真实"服务器的服务器上面，包括使用mod_wsgi来部署到Apache服务器上面。

Bottle会负责URL映射，通过模板来输出页面。待办事项列表被存储在一个SQLite数据库中，通过Python代码来读写数据库。

我们会完成以下页面和功能:


 * 首页 ``http://localhost:8080/todo``
 * 添加待办事项: ``http://localhost:8080/new``
 * 编辑待办事项: ``http://localhost:8080/edit/:no``
 * 通过 @validate 修饰器来验证数据合法性
 * 捕获错误


开始之前...
====================


.. rubric:: 安装

假设你已经安装好了Python (2.5或更改版本)，接下来你只需要下载Bottle就行了。除了Python标准库，Bottle没有其他依赖。

你可通过Python的esay_install命令来安装Bottle: ``easy_install bottle`` 

.. rubric:: 其它软件

因为我们使用SQLite3来做数据库，请确保它已安装。如果是Linux系统，大多数的发行版已经默认安装了SQLite3。SQLite同时可工作在Windows系统和MacOS X系统上面。Pyhton标准库中，已经包含了 `sqlite3` 模块。


.. rubric:: 创建一个SQL数据库

首先，我们需要先创建一个数据库，稍后会用到。在你的项目文件夹执行以下脚本即可，你也可以在Python解释器逐条执行。

::

    import sqlite3
    con = sqlite3.connect('todo.db') # Warning: This file is created in the current directory
    con.execute("CREATE TABLE todo (id INTEGER PRIMARY KEY, task char(100) NOT NULL, status bool NOT NULL)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Read A-byte-of-python',0)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Visit the Python website',1)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Test various editors for syntax highlighting',1)")
    con.execute("INSERT INTO todo (task,status) VALUES ('Choose your favorite WSGI-Framework',0)")
    con.commit()

现在，我们已经创建了一个名字为 `todo.db` 的数据库文件，数据库中有一张名为 ``todo`` 的表，表中有 ``id`` , ``task`` , 及 ``status`` 这三列。每一行的 ``id`` 都是唯一的，稍后会根据id来获取数据。 ``task`` 用于保存待办事项的文本，最大长度为100个字符。最后 ``status`` 用于标明待办事项的状态，0为开启，1为关闭。


基于Bottle的待办事项列表
================================================

为了创建我们的Web应用，我们先来介绍一下Bottle框架。首先，我们需要了解Bottle中的route，即URL映射。


.. rubric:: route URL映射

基本上，浏览器访问的每一页面都是动态生成的。Bottle通过route，将浏览器访问的URL映射到具体的Python函数。例如，在我们访问  ``http://localhost:8080/todo`` 的时候，Bottle会查找 ``todo`` 这个route映射到了哪个函数上面，接着调用该函数来响应浏览器请求。


.. rubric:: 第一步 - 显示所有已开启的待办事项

在我们了解什么是route后，让我们来试着写一个。访问它即可查看所有已开启的待办事项。

::

    import sqlite3
    from bottle import route, run

    @route('/todo')
    def todo_list():
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT id, task FROM todo WHERE status LIKE '1'")
        result = c.fetchall()
        return str(result)

    run()

将上面的代码保存为 ``todo.py`` ，放到 ``todo.db`` 文件所在的目录。如果你想将它们分开放，则需要在 ``sqlite3.connect()`` 函数中写上 ``todo.db`` 文件的路径。

来看看我们写的代码。
1. 导入了必须的 ``sqlite3`` 模块，从Bottle中导入 ``route`` 和 ``run`` 。
2. ``run()`` 函数启动了Bottle的内置开发服务器，默认情况下，开发服务器在监听本地的8080端口。
3. ``route`` 是Bottle实现URL映射功能的修饰器。你可以看到，我们定义了一个 ``todo_list()`` 函数，读取了数据库中的数据。然后我们使用 ``@route('/todo')`` 来将 ``todo_list()`` 函数和``todo`` 这个route绑定在一起。每一次浏览器访问 ``http://localhost:8080/todo`` 的时候，Bottle都会调用 ``todo_list()`` 函数来响应请求，并返回页面，这就是route的工作方式了。

事实上，你可以给一个函数添加多个route。

::

    @route('/todo')
    @route('/my_todo_list')
    def todo_list():
        ...

这样是正确的。但是反过来，你不能将一个route和多个函数绑定在一起。

你在浏览器中看到的即是你在 ``todo_list()`` 函数中返回的页面。在这个例子中，我们通过 ``str()`` 函数将结果转换成字符串，因为Bottle期望函数的返回值是一个字符串或一个字符串的列表。但 `Python DB API`_ 中规定了，数据库查询的返回值是一个元组的列表。

现在，我们已经了解上面的代码是如何工作的，是时候运行它来看看效果了。记得在Linux或Unix系统中， ``todo.py`` 文件需要标记为可执行(译者注：没有必要)。然后，通过 ``python todo.py`` 命令来执行该脚本，接着用浏览器访问 ``http://localhost:8080/todo`` 来看看效果。如果代码没有写错，你应该会在页面看到以下输出。

::

    [(2, u'Visit the Python website'), (3, u'Test various editors for and check the syntax highlighting')]

如果是这样，那么恭喜你！如果出现错误，那么你需要检查代码时候写错，修改完后记得重启HTTP服务器，要不新的版本不会生效。

实际上，这个输出很难看，只是SQL查询的结果。所以，下一步我们会把它变得更好看。


.. rubric:: 调试和自动加载

或许你已经注意到了，如果代码出错的话，Bottle会在页面上显示一个简短的错误信息。例如，连接数据库失败。为了方便调试， 我们希望错误信息更加具体，可加上以下语句。

::

    from bottle import run, route, debug
    ...
    #add this at the very end:
    debug(True)
    run()

开启调试模式后，出错时页面会打印出完整的Python运行栈。另外，在调试模式下，模板也不会被缓存，任何对模板的修改会马上生效，而不用重启服务器。


.. warning::
    ``debug(True)`` 是为开发时的调试服务的， *不应* 在生产环境中开启调试模式。

另外一个十分有用的功能是自动加载，可修改 ``run()`` 语句来开启。

::

    run(reloader=True)
    
这样会自动检测对脚本的修改，并自动重启服务器来使其生效。同上，这个功能并不建议在生产环境中使用。


.. rubric:: 使用模板来格式化输出

现在我们试着格式化脚本的输出，使其更适合查看。

实际上Bottle期望route的回调函数返回一个字符串或一个字符串列表，通过内置的HTTP服务器将其返回给浏览器。Bottle不关心字符串的内容，所以我们可以将其格式化成HTML格式。

Bottle内置了独创的模板引擎。模板是后缀名为 ``.tpl`` 的文本文件。模板的内容混合着HTML标签和Python语句，模板也可以接受参数。例如数据库的查询结果，我们可以在模板内将其漂亮地格式化。

接下来，我们要将数据库的查询结果格式化为一个两列的表格。表格的第一列为待办事项的ID，第二列为待办事项的内容。查询结果是一个元组的列表，列表中的每个元组后包含一个结果。

在例子中使用模板，只需要添加以下代码。

::

    from bottle import route, run, debug, template
    ...
    result = c.fetchall()
    c.close()
    output = template('make_table', rows=result)
    return output
    ...

我们添加了两样东西。首先我们从Bottle中导入了 ``template`` 函数以使用模板功能，接着，我们渲染 ``make_table`` 这个模板(参数是rows=result)，把模板函数的返回值赋予 ``output`` 变量，并返回 ``output`` 。如有必要，我们可添加更多的参数。

模板总是返回一个字符串的列表，所以我们无须转换任何东西。当然，我们可以将返回写为一行以减少代码量。

::

 return template('make_table', rows=result)
 
对应的模板文件。

::

    %#template to generate a HTML table from a list of tuples (or list of lists, or tuple of tuples or ...)
    <p>所有已开启的待办事项:</p>
    <table border="1">
    %for row in rows:
      <tr>
      %for col in row:
        <td>{{col}}</td>
      %end
      </tr>
    %end
    </table>

将上面的代码保存为 ``make_table.tpl`` 文件，和 ``todo.py`` 放在同一个目录。

看看上面的代码，以%开头的行被当作Python代码来执行。请注意，只有正确的Python语句才能通过编译，要不模板就会抛出一个异常。除了Python语句，其它都是普通的HTML标记。

如你所见，为了遍历 ``rows`` ，我们两次使用了Python的 ``for`` 语句。 ``rows``是持有查询结果的变量，一个元组的列表。第一个 ``for`` 语句遍历了列表中所有的元组，第二个 ``for`` 语句遍历了元组中的元素，将其放进表格中。 ``for`` , ``if`` , ``while`` 语句都需要通过 ``%end`` 来关闭，要不会得到不正确的结果。

如果想要在不以%开头的行中访问变量，则需要把它放在两个大括号中间。这告诉模板，需要用变量的实际值将其替换掉。

再次运行这个脚本，页面输出依旧不是很好看，但是更具可读性了。当然，你可给模板中的HTML标签加上CSS样式，使其更好看。


.. rubric:: 使用GET和POST

能够查看所有代码事项后，让我们进入到下一步，添加新的待办事项到列表中。新的待办事项应该在一个常规的HTML表单中，通过GET方式提交。

让我们先来添加一个接受GET请求的route。

::

    from bottle import route, run, debug, template, request
    ...
    return template('make_table', rows=result)
    ...

    @route('/new', method='GET')
    def new_item():

        new = request.GET.get('task', '').strip()

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()

        c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
        new_id = c.lastrowid

        conn.commit()
        c.close()

        return '<p>新的待办事项已添加到数据库中,ID为 %s</p>' % new_id

为了访问GET(或POST)中的数据，我们需要从Bottle中导入 ``request`` ，通过 ``request.GET.get('task', '').strip()`` 来获取表单中 ``task`` 字段的数据。可多次使用 ``request.GET.get()`` 来获取表单中所有字段的数据。
 
接下来是对数据的操作：写入数据库，获取返回的ID，生成页面。

因为我们是从HTML表单中获取数据，所以现在让我们来创建这个表单吧。我们通过 ``/new`` 这个URL来添加待办事项。

::

    ...
    @route('/new', method='GET')
    def new_item():

    if request.GET.get('save','').strip():

        new = request.GET.get('task', '').strip()
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()

        c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
        new_id = c.lastrowid

        conn.commit()
        c.close()

        return '<p>新的待办事项已添加到数据库中,ID为 %s</p>' % new_id
    else:
        return template('new_task.tpl')


对应的 ``new_task.tpl`` 模板如下。

::

    <p>添加一个待办事项:</p>
    <form action="/new" method="GET">
    <input type="text" size="100" maxlength="100" name="task">
    <input type="submit" name="save" value="save">
    </form>

如你所见，这个模板只是纯HTML的，不包含Python代码。这样，我们就完成了添加待办事项这个功能。如果你想通过POST来获取数据，那么用 ``request.POST.get()`` 来代替 ``request.GET.get()`` 就行了。


.. rubric:: 修改已有待办事项

最后，我们需要做的是修改已有待办事项。

仅使用我们当前了解到的route类型，是可以完成这个任务的，但太取巧了。Bottle还提供了一种 ``动态route`` ，可以更简单地实现。

基本的动态route声明如下::

    @route('/myroute/:something')

关键的区别在于那个冒号。它告诉了Bottle，在下一个 ``/`` 之前， ``:something`` 可以匹配任何字符串。 ``:something`` 匹配到的字符串会传递给回调函数，进一步地处理。

在我们的待办事项应用里，我们创建一个route( ``@route('edit/:no')`` ), ``no`` 是待办事项在数据库里面的ID。

对应的代码如下。

::

    @route('/edit/:no', method='GET')
    def edit_item(no):

        if request.GET.get('save','').strip():
            edit = request.GET.get('task','').strip()
            status = request.GET.get('status','').strip()

            if status == 'open':
                status = 1
            else:
                status = 0

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("UPDATE todo SET task = ?, status = ? WHERE id LIKE ?", (edit, status, no))
            conn.commit()

            return '<p>ID为%s的待办事项已更新</p>' % no
        else:
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (str(no)))
            cur_data = c.fetchone()

            return template('edit_task', old=cur_data, no=no)

这和之前的添加待办事项类似，主要的不同点在于使用了动态的route( ``:no`` )，它可将ID传给route对应的回调函数。如你所见，我们在 ``edit_item`` 函数中使用了 ``no`` ，从数据库中获取数据。

对应的 ``edit_task.tpl`` 模板如下。

::

    %#template for editing a task
    %#the template expects to receive a value for "no" as well a "old", the text of the selected ToDo item
    <p>Edit the task with ID = {{no}}</p>
    <form action="/edit/{{no}}" method="get">
    <input type="text" name="task" value="{{old[0]}}" size="100" maxlength="100">
    <select name="status">
    <option>open</option>
    <option>closed</option>
    </select>
    <br/>
    <input type="submit" name="save" value="save">
    </form>

再一次，模板中混合了HTML代码和Python代码，之前已解释过。

你也可在动态route中使用正则表达式，稍后会提及。

.. rubric:: 验证动态route

在某些场景下，需要验证route中的可变部分。例如，在上面的例子中，我们的 ``no`` 需要是一个整形数，如果我们的输入是一个浮点数，或字符串，Python解释器将会抛出一个异常，这并不是我们想要的结果。

对应上述情况，Bottle提供了一个 ``@validate`` 修饰器，可在用户输入被传递给回调函数之前，检验用户数据的合法性。代码例子如下。

::

    from bottle import route, run, debug, template, request, validate
    ...
    @route('/edit/:no', method='GET')
    @validate(no=int)
    def edit_item(no):
    ...

首先，我们从Bottle中导入了 ``validate`` ，然后在route中使用了。在这里，我们验证 ``no`` 是否是一个整形数。基本上， ``validate`` 可用于其它类型，例如浮点数，列表等等。

保存更改，如果用户提供的 ``:no`` 不是一个整形数，而是一个浮点数或其他类型，将返回一个"403 forbidden"页面，而不是抛出异常。

.. rubric:: 在动态route中使用正则表达式

Bottle允许在动态route中使用正则表达式。

我们假设需要通过 ``item1`` 这样的形式来访问数据库中id为1的待办事项。显然，我们不想为每个待办事项都创建一个route。鉴于route中的"item"部分是固定的，简单的route就无法满足需求了，我们需要在route中使用正则表达式。

使用正则表达式的解决方法如下。

::

    @route('/item:item#[1-9]+#')
    def show_item(item):
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (item))
        result = c.fetchall()
        c.close()
        if not result:
            return '该待办事项不存在!'
        else:
            return '待办事项: %s' %result[0]

当然，这个例子是我们想象出来的，去掉"item1"中的"item"，直接使用"1"会更简单。虽然如此，我们还是想为你展示在route的正则表达式: ``@route(/item:item_#[1-9]+#)`` 和一个普通的route差不多，但是在两个"#"中的字符就是一个正则表达式，匹配从0到9的数字。在处理"/item9"这样的请求的时候，正则表达式会匹配到"9"，然后将"9"做为item参数传递给show_item函数，而不是"item9"。

.. rubric:: 返回静态文件

有时候，我们只是想返回已有的静态文件。例如我们的应用中有个静态的帮助页面help.html，我们不希望每次访问帮助页面的时候都动态生成。

::

    from bottle import route, run, debug, template, request, validate, static_file

    @route('/help')
    def help():
        return static_file('help.html', root='/path/to/file')

首先，我们需要从Bottle中导入 ``static_file`` 函数。它接受至少两个参数，一个是需要返回的文件的文件名，一个是该文件的路径。即使该文件和你的应用在同一个目录下，还是要指定文件路径(可以使用".")。Bottle会猜测文件的MIME类型，并自动设置。如果你想显式指定MIME类型，可以在static_file函数里面加上例如 ``mimetype='text/html'`` 这样的参数。 ``static_file`` 函数可和任何route配合使用，包括动态route。


.. rubric:: 返回JSON数据

有时我们希望返回JSON，以便在客户端使用JavaScript来生成页面，Bottle直接支持返回JSON数据了。

我们假设现在需要返回JSON数据。

::

    @route('/json:json#[1-9]+#')
    def show_json(json):
        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (json))
        result = c.fetchall()
        c.close()

        if not result:
            return {'task':'对应的待办事项不存在'}
        else:
            return {'Task': result[0]}

很简单，只需要返回一个Python中的字典就可以了，Bottle会自动将其转换为JSON，再传输到客户端。如果你访问"http://localhost/json1"，你应能得到 ``{"Task": ["Read A-byte-of-python to get a good introduction into Python"]}`` 类型的JSON数据。



.. rubric:: 捕获错误

为了避免用户看到出错信息，我们需要捕获应用运行时出现的错误，以提供更友好的错误提示。Bottle提供了专门用于捕获错误的route。

例如，我们想捕获403错误。

::

    from bottle import error

    @error(403)
    def mistake(code):
        return '参数格式错误！'

首先，我们需要从Bottle中导入 ``error`` ，然后通过 ``error(403)`` 来定义创建一个route，用于捕获所有"403 forbidden"错误。注意，该route总是会将error-code传给 ``mistake()`` 函数，即使你不需要它。所以回调函数至少要接受一个参数，否则会失效。

一样的，同一个回调函数可以捕获多种错误。

::

    @error(404)
    @error(403)
    def mistake(code):
        return '出错啦！'

效果和下面一样。

::

    @error(403)
    def mistake403(code):
        return '参数格式错误！'

    @error(404)
    def mistake404(code):
        return '该页面不存在！'


.. rubric:: 总结

通过以上章节，你应该对Bottle框架有了一个大致的了解，可以使用Bottle进行开发了。

接下来的章节会简单介绍一下，如何在大型项目中使用Bottle。此外，我们还会介绍如何将Bottle部署到更高性能的Web服务器上。


部署服务器
================================

到目前为止，我们还是使用Bottle内置的，随Python一起发布的 `WSGI reference Server`_ 服务器。尽管该服务器十分适合用于开发环境，但是它确实不适用于大项目。在我们介绍其他服务器之前，我们先看看如何优化内置服务器的设置。

.. rubric:: 更改服务器的端口和IP

默认的，Bottle会监听127.0.0.1(即 ``localhost`` )的 ``8080`` 端口。如果要更改该设置，更改 ``run`` 函数的参数即可。

更改端口，监听80端口

::

    run(port=80)

更改IP地址

::

    run(host='123.45.67.89')

可同时使用

::

   run(port=80, host='123.45.67.89')

当Bottle运行在其他服务器上面时， ``port`` 和 ``host`` 参数依然适用，稍后会介绍。


.. rubric:: 在其他服务器上运行

在大型项目上，Bottle自带的服务器会成为一个性能瓶颈，因为它是单线程的，一次只能响应一个请求。Bottle已经可以工作在很多多线程的服务器上面了，例如 Cherrypy_, Fapws3_, Flup_ 和 Paste_ ，所以我们建议在大型项目上使用高性能的服务器。

如果想运行在Paste服务器上面，代码如下(译者注：需要先安装Paste)。

::

    from bottle import PasteServer
    ...
    run(server=PasteServer)

其他服务器如 ``FlupServer``, ``CherryPyServer`` 和 ``FapwsServer`` 也类似。


.. rubric:: 使用 mod_wsgi_ 运行在Apache上

或许你已经有了一个 Apache_ 服务器，那么可以考虑使用 mod_wsgi_ 。

我们假设你的Apache已经能跑起来，且mod_wsgi也能工作了。在很多Linux发行版上，都能通过包管理软件简单地安装mod_wsgi。

Bottle已经自带用于mod_wsgi的适配器，所以让Bottle跑在mod_wsgi上面是很简单的。

接下来的例子里，我们假设你希望通过 ``http://www.mypage.com/todo`` 来访问"ToDo list"这个应用，且代码、模板、和SQLite数据库存放在 ``/var/www/todo`` 目录。

如果通过mod_wsgi来运行你应用，那么必须从代码中移除 ``run()`` 函数。

然后，创建一个 ``adapter.wsgi`` 文件，内容如下。

::

    import sys, os, bottle

    sys.path = ['/var/www/todo/'] + sys.path
    os.chdir(os.path.dirname(__file__))

    import todo # This loads your application

    application = bottle.default_app()

将其保存到 ``/var/www/todo`` 目录下面。其实，可以给该文件起任何名字，只要后缀名为 ``.wsgi`` 即可。

最后，我们需要在Apache的配置中添加一个虚拟主机。

::

    <VirtualHost *>
        ServerName mypage.com

        WSGIDaemonProcess todo user=www-data group=www-data processes=1 threads=5
        WSGIScriptAlias / /var/www/todo/adapter.wsgi

        <Directory /var/www/todo>
            WSGIProcessGroup todo
            WSGIApplicationGroup %{GLOBAL}
            Order deny,allow
            Allow from all
        </Directory>
    </VirtualHost>

重启Apache服务器后，即可通过 ``http://www.mypage.com/todo`` 来访问你的应用。


结语
=========================

现在，我们这个教程已经结束了。我们学习了Bottle的基础知识，然后使用Bottle来写了第一个应用。另外，我们还介绍了如何在大型项目中使用Bottle，以及使用mod_wsgi在Apache中运行Bottle应用。

我们并没有在这份教程里介绍Bottle的方方面面。我们没有介绍如何上传文件，验证数据的可靠性。
还有，我们也没介绍如何在模板中调用另一个模板。以上，可以在 `Bottle documentation`_ 中找到答案。


完整代码
=========================

译者注：以下内容不翻译

``todo.py`` 文件
 
::

    import sqlite3
    from bottle import route, run, debug, template, request, validate, static_file, error

    # only needed when you run Bottle on mod_wsgi
    from bottle import default_app

    @route('/todo')
    def todo_list():

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT id, task FROM todo WHERE status LIKE '1';")
        result = c.fetchall()
        c.close()

        output = template('make_table', rows=result)
        return output

    @route('/new', method='GET')
    def new_item():

        if request.GET.get('save','').strip():

            new = request.GET.get('task', '').strip()
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()

            c.execute("INSERT INTO todo (task,status) VALUES (?,?)", (new,1))
            new_id = c.lastrowid

            conn.commit()
            c.close()

            return '<p>The new task was inserted into the database, the ID is %s</p>' % new_id

        else:
            return template('new_task.tpl')

    @route('/edit/:no', method='GET')
    @validate(no=int)
    def edit_item(no):

        if request.GET.get('save','').strip():
            edit = request.GET.get('task','').strip()
            status = request.GET.get('status','').strip()

            if status == 'open':
                status = 1
            else:
                status = 0

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("UPDATE todo SET task = ?, status = ? WHERE id LIKE ?", (edit,status,no))
            conn.commit()

            return '<p>The item number %s was successfully updated</p>' %no

        else:
            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (str(no)))
            cur_data = c.fetchone()

            return template('edit_task', old = cur_data, no = no)

    @route('/item:item#[1-9]+#')
    def show_item(item):

            conn = sqlite3.connect('todo.db')
            c = conn.cursor()
            c.execute("SELECT task FROM todo WHERE id LIKE ?", (item))
            result = c.fetchall()
            c.close()

            if not result:
                return 'This item number does not exist!'
            else:
                return 'Task: %s' %result[0]

    @route('/help')
    def help():

        static_file('help.html', root='.')

    @route('/json:json#[1-9]+#')
    def show_json(json):

        conn = sqlite3.connect('todo.db')
        c = conn.cursor()
        c.execute("SELECT task FROM todo WHERE id LIKE ?", (json))
        result = c.fetchall()
        c.close()

        if not result:
            return {'task':'This item number does not exist!'}
        else:
            return {'Task': result[0]}


    @error(403)
    def mistake403(code):
        return 'There is a mistake in your url!'

    @error(404)
    def mistake404(code):
        return 'Sorry, this page does not exist!'


    debug(True)
    run(reloader=True)
    #remember to remove reloader=True and debug(True) when you move your application from development to a productive environment

Template ``make_table.tpl``::

    %#template to generate a HTML table from a list of tuples (or list of lists, or tuple of tuples or ...)
    <p>The open items are as follows:</p>
    <table border="1">
    %for row in rows:
      <tr>
      %for col in row:
        <td>{{col}}</td>
      %end
      </tr>
    %end
    </table>

Template ``edit_task.tpl``::

    %#template for editing a task
    %#the template expects to receive a value for "no" as well a "old", the text of the selected ToDo item
    <p>Edit the task with ID = {{no}}</p>
    <form action="/edit/{{no}}" method="get">
    <input type="text" name="task" value="{{old[0]}}" size="100" maxlength="100">
    <select name="status">
    <option>open</option>
    <option>closed</option>
    </select>
    <br/>
    <input type="submit" name="save" value="save">
    </form>

Template ``new_task.tpl``::

    %#template for the form for a new task
    <p>Add a new task to the ToDo list:</p>
    <form action="/new" method="GET">
    <input type="text" size="100" maxlength="100" name="task">
    <input type="submit" name="save" value="save">
    </form>
