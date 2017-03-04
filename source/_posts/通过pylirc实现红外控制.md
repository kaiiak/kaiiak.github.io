title: 通过pylirc实现红外控制
date: 2015-11-14 20:39:40
tags: [pylirc,树莓派]
---

## 官方文档
在2005年更新的官方文档，点击[这里](http://bazaar.launchpad.net/~rockstar/pylirc2/trunk/view/head:/doc/simple.txt).

```doc
pyLirc v0.0.5

Introduction

pyLirc is a module for Python that interacts with lirc to give 
Python programs the ability to receive commands from remote 
controls.

This isn't much of documentation, but at least it's a start and
there isn't much to document right now anyway.


Initialization

Before you can receive any commands from lirc, you'll need to 
initialize the module. After importing pyLirc, call the pylirc.init()
function:

   import pylirc

   integer = pylirc.init(string name[, string configuration [, integer blocking ]])

the returnvalue is the returnvalue of lircs client library
lirc_init(), ie a socket, or zero on failure.

The socket can be used with select.select() to wait for data if you don't
want to use blocking. This is especially useful in multithreaded programs
as blocking mode of pylirc will blick all threads, whereas select() will
only block the current and with optional timeout.

name: the name used for your program in the lirc configuration
file, must be supplied.

configuration:  a filename to a lirc configuration file in case you wish not to
use lircs default configuration file (usually ~/.lircrc).

blocking: a flag indicating wether you want blocking mode or not. See also 
blocking() and select.select() (latter in python docs)


Polling

If initialization was ok, you can poll lirc for commands. To read any commands
in queue call pylirc.nextcode():

   list = pylirc.nextcode([integer Exteneded])

The returnvalue is 'None', if no commands was on the queue, or a list
containing the commands read.

To get the commands one by one enumerate the list:

   for code in list:
      print code

If you supply the optional argument Extended as true, code will be a dictionary
otherwise it will be a string (old behaviour).

The dictionary currently contains:
"config": The config string from lirc config file - the same string you'd get in
          non-extended mode.
"repeat": The repeat count of the buttonpress.
      
Note, that there can still be more commands on queue after a call
to pylirc.nextcode(). You should call it in a loop until you get
'None' back.


Exiting

When you're done using pyLirc and before you exit your program you
should clean up:

   pylirc.exit()


Changing mode

When you initialize pyLirc, you can chose wether you want blocking or
non-blocking mode. Blocking mode means pylirc.nextcode() waits until
there is a command to be read until it returns.
To change mode after initialization, use blocking():

   success = pylirc.blocking(int)
```

## 中文翻译
自己尝试翻译一下，英语渣渣，有不正确的地方请指正。

```doc
pyLirc v0.0.5

介绍

pyLirc是一个与LIRC交互给
Python程序从远程接收远程命令的能力
的Python模块。

这是没有太多的文档，毕竟它是一个新项目，
现在反正没有太多的文档。

初始化

在接受来自lirc的命令之前，你应该初始化本模块。在 import pylirc之后，使用 pylirc.inti()函数：
	import pylirc
	integer = pylirc.init(string name[, string configuration [, integer blocking ]])
返回值是客户端库中lirc_init()函数的返回值，是一个socket，如果返回值是0，则初始化失败。


```

# 示例代码

```python
#!/usr/bin/env python
import pylirc

class Buttons:
    SELECT = 0
    RIGHT = 1
    DOWN = 2
    UP = 3
    LEFT = 4

    def __init__(self, app, conf):
        if not pylirc.init(app, conf, 1):
            raise Exception("Unable to init pylirc");
        pylirc.blocking(0)

    def readButton(self):
        btn = pylirc.nextcode()
        if btn:
            return btn[0]
        else:
            return None
```