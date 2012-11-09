Bottle中文文档 0.11dev
===================================

**注意：为了更方便地保持和官方文档的同步，我新建了一个项目，在 https://github.com/myzhan/bottle 下。这个fork不再维护。**

**新的项目中，中文文档的po文件在docs/_locale/zh_CN目录下，可利用sphinx的i18n机制生成中文文档。**

已预先生成了bottle-zh-cn.pdf文件，原始的rst文件在docs目录下。

fork自bottle项目，由于历史原因，现在新建一个文档翻译项目似乎更方便，不容易冲突。

安装了完整texlive环境的话，可以自己生成pdf。

在docs目录执行
make latex

然后去../build/docs/latex/下面执行
xelatex Bottle.tex

sphinx默认的latex命令不支持中文，暂时我只有这个办法。
