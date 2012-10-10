开发者笔记
=================

这份文档是为那些对Bottle的开发和发布流程感兴趣的开发者和软件包维护者准备的。如果你想要做贡献，看它是对的！


参与进来
------------

有多种加入社区的途径，保持消息的灵通。这里是一些方法:

* **邮件列表**: 发送邮件到 `bottlepy+subscribe@googlegroups.com <mailto:bottlepy+subscribe@googlegroups.com>`_ 以加入我们的邮件列表(无需Google账户)
* **Twitter**: 在 `Twitter <https://twitter.com/bottlepy>`_ 上关注我们或搜索 `bottlepy <https://twitter.com/#!/search/%23bottlepy>`_ 标签
* **IRC**: 加入 `#irc.freenode.net上的bottlepy频道 <irc://irc.freenode.net/bottlepy>`_ 或使用 `web聊天界面 <http://webchat.freenode.net/?channels=bottlepy>`_ 
* **Google plus**: 有时我们会在Google+页面上发表一些 `与Bottle有关的博客 <https://plus.google.com/b/104025895326575643538/104025895326575643538/posts>`_ 


获取源代码
---------------

Bottle的 `开发仓库 <https://github.com/defnull/bottle>`_ 和 `问题追踪 <https://github.com/defnull/bottle/issues>`_ 都搭建在 `github <https://github.com/defnull/bottle>`_ 上面。如果你打算做贡献，创建一个github帐号然后fork一下吧。你做出的更改或建议都可被其他开发者看到，可以一起讨论。即使没有github帐号，你也可以clone代码仓库，或下载最新开发版本的压缩包。


* **git:** ``git clone git://github.com/defnull/bottle.git``
* **git/https:** ``git clone https://github.com/defnull/bottle.git``
* **Download:** Development branch as `tar archive <http://github.com/defnull/bottle/tarball/master>`_ or `zip file <http://github.com/defnull/bottle/zipball/master>`_.


发布和更新
--------------------

Bottle不定时发布正式版本，通过 `PyPi <http://pypi.python.org/pypi/bottle>`_ 来分发。RC版或bugfix版本只会在github上面发布。一些Linux发行版也许会提供一些过时的版本。

Bottle的版本号分隔为三个部分(**major.minor.revision**)。版本号 *不会* 用于引入新特征，只是为了指出重要的bug-fix或API的改动。关键的bug会在近两个新的版本中修复，然后通过所有渠道发布(邮件列表, twitter, github)。不重要的bug修复或特征不保证添加到旧版本中。这个情况在未来也许会改变。

Major 发布版本 (x.0)
    在重要的里程碑达成或新的更新和旧版本彻底不兼容时，会增加major版本号。为了使用新版本，你也许需要修改整个应用，但major版本号极少会改变。

Minor 发布版本 (x.y)
    在更改了API之后，或增加minor版本号。你可能会收到一些API已过时的警告，调整设置以继续使用旧的特征，但在大多数情况下，这些更新都会对旧版本兼容。你应当保持更新，但这不是必须的。0.8版本是一个特例，它 *不兼容* 旧版本(所以我们跳过了0.7版本直接发布0.8版本)，不好意思。

Revision 发布版本(x.y.z)
    在修复了一些bug，和改动不会修改API的时候，会增加revision版本号。你可以放心更新，而不用修改你应用的代码。事实上，你确实应该更新这类版本，因为它常常修复一些安全问题。

Pre-Release 发布版本
    RC版本会在版本号中添加 ``rc`` 字样。API已基本稳定，已经开放测试，但还没正式发布。你不应该在生产环境中使用rc版本。


代码仓库结构
--------------------

代码仓库的结构如下:

``master`` 分支
  该分支用于集成，测试和开发。所有计划添加到下一版本的改动，会在这里合并和测试。

``release-x.y`` 分支
  只要master分支已经可以用来发布一个新的版本，它会被安排到一个新的发行分支里面。在RC阶段，不再添加或删除特征，只接受bug-fix和小改动，直到认为它可以用于生产环境和正式发布。基于这点考虑，我们称之为“支持分支(support branch)”，依然接受bug-fix，但仅限关键的bug-fix。每次更改后，版本号都会更新，这样你就能及时获取重要的改动了。

``bugfix_name-x.y`` 分支
  这些分支是临时性的，用于修复现有发布版本的bug。在合并到其他分支后，它们就会被删除。

特征分支
  所有这类分支都是用于新增特征的。基于master分支，在开发、合并完毕后，就会被删除。


.. rubric:: 对于开发者，这意味着什么？

如果你想添加一个特征，可以从 ``master`` 分支创建一个分支。如果你想修复一个bug，可从 ``release-x.y`` 这类分支创建一个分支。无论是添加特征还是修复bug，都建议在一个独立的分支上进行，这样合并工作就简单了。就这些了！在页面底部会有git工作流程的例子。

Oh，请不要修改版本号。我们会在集成的时候进行修改。因为你不知道我们什么时候会将你的request pull下来:)


.. rubric:: 对于软件包维护者，这意味着什么？

关注那些bugfix和新版本的tag，还有邮件列表。如果你想从代码仓库中获取特定的版本，请使用tag，而不是分支。分支中也许会包含一些未发布的改动，但tag会标记是那个commit更改了版本号。


提交补丁
------------------

让你的补丁被集成进来的最好方法，是在github上面fork整个项目，创建一个新的分支，修改代码，然后发送一个pull-request。页面下方是git工作流程的例子，也许会有帮助。提交git兼容的补丁文件到邮件列表也是可以的。无论使用什么方法，请遵守以下的基本规则:

* **文档：** 告诉我们你的补丁做了什么。注释你的代码。如果你添加了新的特征，请添加相应的使用方法。
* **测试：** 编写测试以证明你的补丁如期工作，且没有破坏任何东西。如果你修复了一个bug，至少写一个测试用例来触发这个bug。在提交补丁之前，请确保所有测试已通过。
* **一次只提交一个补丁：** 一次只修改一个bug，一次只添加一个新特征。保持补丁的干净。
* **与上流同步：** 如果在你编写补丁的时候， ``upstream/master`` 分支被修改了，那么先rebase或将新的修改pull下来，确保你的补丁在合并的时候不会造成冲突。


生成文档
--------------------------

你需要一个Sphinx的新版本来生产整份文档。建议将Sphinx安装到一个 :command:`virtualenv` 环境。

.. code-block:: bash

  # Install prerequisites
  which virtualenv || sudo apt-get install python-virtualenv
  virtualenv --no-site-dependencies venv
  ./venv/pip install -U sphinx

  # Clone or download bottle from github
  git clone https://github.com/defnull/bottle.git

  # Activate build environment
  source ./venv/bin/activate

  # 生成HTML格式的文档
  cd bottle/docs
  make html

  # 可选：安装生成PDF所需软件包
  sudo apt-get install texlive-latex-extra \
                       texlive-latex-recommended \
                       texlive-fonts-recommended

  # 可选：生成PDF文件
  make latex
  cd ../build/docs/latex
  make pdf


GIT工作流程
---------------------

接下来的例子都假设你已经有一个 `git的免费帐号 <https://github.com>`_ 。虽然不是必须的，但可简单化很多东西。

首先，你需要从官方代码仓库创建一个fork。只需在 `bottle项目页面 <https://github.com/defnull/bottle>`_ 点击一下"fork"按钮就行了。创建玩fork之后，会得到一个关于这个新仓库的简介。

你刚刚创建的fork托管在github上面，对所有人都是可见的，但只有你有修改的权限。现在你需要将其从线上clone下面，做出实际的修改。确保你使用的是(可写-可读)的私有URL，而不是(只读)的公开URL。

::

  git clone git@github.com:your_github_account/bottle.git

在你将代码仓库clone下来后，就有了一个"origin"分支，指向你在github上的fork。不要让名字迷惑了你，它并不指向bottle的官方代码仓库，只是指向你自己的fork。为了追踪官方的代码仓库，可添加一个新的"upstream"远程分支。

::

  cd bottle
  git remote add upstream git://github.com/defnull/bottle.git
  git fetch upstream

注意，"upstream"分支使用的是公开的URL，是只读的。你不能直接往该分支push东西，而是由我们来你的公开代码仓库pull，后面会讲到。

.. rubric:: 提交一个特征

在独立的特征分支内开发新的特征，会令集成工作更简单。因为它们会被合并到 ``master`` 分支，所有它们必须是基于 ``upstream/master`` 的分支。下列命令创建一个特征分支。

::

  git checkout -b cool_feature upstream/master
  
现在可开始写代码，写测试，更新文档。在提交更改之前，记得确保所有测试已经通过。

::

  git commit -a -m "Cool Feature"

与此同时，如果 ``upstream/master`` 这个分支有改动，那么你的提交就有可能造成冲突，可通过rebase操作来解决。

::

  git fetch upstream
  git rebase upstream

这相当于先撤销你的所有改动，更新你的分支到最新版本，再重做你的所有改动。如果你已经发布了你的分支(下一步会提及)，这就不是一个好主意了，因为会覆写你的提交历史。这种情况下，你应该先将最新版本pull下来，手动解决所有冲突，运行测试，再提交。

现在，你已经做好准备发一个pull-request了。但首先你应该公开你的特征分支，很简单，将其push到你github的fork上面就行了。

::

  git push origin cool_feature

在你push完你所有的commit之后，你需要告知我们这个新特征。一种办法是通过github发一个pull-request。另一种办法是把这个消息发到邮件列表，这也是我们推荐的方式，这样其他开发者就能看到和讨论你的补丁，你也能免费得到一些反馈 :)

如果我们接受了你的补丁，我们会将其集成到官方的开发分支中，它将成为下个发布版本的一部分。

.. rubric:: 修复Bug

修复Bug和添加一个特征差不多，下面是一些不同点:

1) 修复所有受影响的分支，而不仅仅是开发分支(Branch off of the affected release branches instead of just the development branch)。
2) 至少编写一个触发该Bug的测试用例。
3) 修复所有受影响的分支，包括 ``upstream/master`` ，如果它也受影响。 ``git cherry-pick`` 可帮你完成一些重复工作。
4) 名字后面要加上其修复的版本号，以防冲突。例子: ``my_bugfix-x.y`` 或 ``my_bugfix-dev`` 。


