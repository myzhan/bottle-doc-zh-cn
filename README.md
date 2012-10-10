Bottle中文文档 0.11dev
===================================

已预先生成了bottle-zh-cn.pdf文件，原始的rst文件在docs目录下。

fork自bottle项目，由于历史原因，现在新建一个文档翻译项目似乎更方便，不容易冲突。

安装了完整texlive环境的话，可以自己生成pdf。

在docs目录执行
make latex

然后去../build下面执行
xelatex Bottle.tex

sphinx默认的latex命令不支持中文，暂时我只有这个办法。
