---
title: Python logging日志模块使用小结
date: 2016-10-18 15:28:21
tags:
- Python
- Test
---
在运行程序时，为了监控程序的行为需要对程序行为记录日志。

在Python 下，就有这样一个专门用来做日志记录的类库，logging

## 简单的将日志打印到屏幕
```python
#!/usr/bin/env python
# -*- coding: utf8 -*-

# -- modules --
import logging

# -- start --
if __name__ == '__main__':
    # 默认情况下，logging将日志打印到屏幕，日志级别为WARNING

    # 日志级别大小关系为：CRITICAL > ERROR
    # > WARNING > INFO > DEBUG > NOTSET

    logging.debug('DEBUG') # 屏幕不显示
    logging.info('INFO') # 屏幕不显示
    logging.warning('WARNING')
```
屏幕输出
```
wang@Wang:~/Workspace/Python/Template/logger$ ./logger1.py
WARNING:root:WARNING
通过logging.basicConfig 函数对日志的输出格式及方式做相关配置
#!/usr/bin/env python
# -*- coding: utf8 -*-

# -- modules --
import logging

# -- start --
if __name__ == '__main__':
    logging.basicConfig(level=logging.DEBUG,
    format='[%(asctime)s] %(filename)s [line:%(lineno)d] [%(levelname)s] %(message)s',
            datefmt='%a, %d %b %Y %H:%M:%S',
            filename='test.log',
            filemode='w')
    logging.exception('This is EXCEPTION')
    try:
        1/0
    except Exception, e:
        logging.exception('This is EXCEPTION [%s]' % e)
        logging.critical('This is CRITICAL')
        logging.error('This is ERROR')
        logging.warning('This is WARNING')
        logging.info('This is INFO')
        logging.debug('This is DEBUG')
```
配置完成后，日志输出至test.log 日志文件

**注意：由于日志写入模式设置为’w '，因此重复运行时会将之前的日志清空。**

logging.basicConfig 函数各参数:
```
filename: 指定日志文件名
filemode: 和file函数意义相同，指定日志文件的打开模式，’w’或’a’
format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
%(levelno)s: 打印日志级别的数值
%(levelname)s: 打印日志级别名称
%(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
%(filename)s: 打印当前执行程序名
%(funcName)s: 打印日志的当前函数
%(lineno)d: 打印日志的当前行号
%(asctime)s: 打印日志的时间
%(thread)d: 打印线程ID
%(threadName)s: 打印线程名称
%(process)d: 打印进程ID
%(message)s: 打印日志信息
datefmt: 指定时间格式，同time.strftime()
level: 设置日志级别，默认为logging.WARNING
stream: 指定将日志的输出流，可以指定输出到sys.stderr,
sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略
```
日志输出内容如下：
```
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:14] [ERROR] This is EXCEPTION
None
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:18] [ERROR] This is EXCEPTION [integer division or modulo by zero]
Traceback (most recent call last):
File "./logger2.py", line 16, in <module>
1/0
ZeroDivisionError: integer division or modulo by zero
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:19] [CRITICAL] This is CRITICAL
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:20] [ERROR] This is ERROR
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:21] [WARNING] This is WARNING
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:22] [INFO] This is INFO
[Mon, 15 Apr 2013 22:37:28] logger2.py [line:23] [DEBUG] This is DEBUG
```
特别指出EXCEPTION

在日志中输出为ERROR ，但是会多输出一行错误信息，如果没有设置，则输出None 。如日志第一条

EXCEPTION 的正确用法是嵌套在except:  中，并用以输出错误信息。

要指出的是，错误信息会为EXCEPTION 保持，即使在logging.debug() 后logging.exception(‘X’)

那么仍会输出原来未改变的错误信息，即输出
```
[Mon, 15 Apr 2013 xx:xx:xx] logger2.py [line:18] [ERROR] X [integer division or modulo by zero]
Traceback (most recent call last):
File "./logger2.py", line 16, in <module>
1/0
```
将日志同时输出到文件和屏幕
```python
#!/usr/bin/env python
# -*- coding: utf8 -*-

# -- modules --
import logging

# -- start --
if __name__ == '__main__':
    logging.basicConfig(level=logging.DEBUG,
    format='%(asctime)s %(filename)s [line:%(lineno)d] %(levelname)s %(message)s',
            datefmt='%a, %d %b %Y %H:%M:%S',
            filename='test.log',
            filemode='w')

    console = logging.StreamHandler()
    console.setLevel(logging.WARNING)
    formatter=logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
    console.setFormatter(formatter)
    logging.getLogger('').addHandler(console)

    logging.debug('This is DEBUG')
    logging.info('This is INFO')
    logging.warning('This is WARNING')
```
注意：basicConfig中设置的为写入文件的日志级别，但是在屏幕输出的内容级别需要额外设置

屏幕输出为：
```
root : WARNING This is WARNING
日志文件输出为：

Tue, 16 Apr 2013 16:57:21 logger3.py [line:21] DEBUG This is DEBUG
Tue, 16 Apr 2013 16:57:21 logger3.py [line:22] INFO This is INFO
Tue, 16 Apr 2013 16:57:21 logger3.py [line:23] WARNING This is WARNING
```
## logging之日志回滚
略，没看懂。。

## 通过logging.config模块配置日志
```shell
# --- logconf.ini -----------------------------------------------------------
# "loggers"模块包含logger配置的所有关键字，这些关键字并非是直接的配置文件模块名
# 但是，却是Python脚本调用时会使用到的名字。
# 单独的模块名命名为"logger_xxx" 其中"xxx"部分为"key"
# 例如 "logger_root", "logger_log02"

[loggers]
keys=root,log02,log03,log04,log05,log06,log07

# "handlers" 模块包含所有的 handler 配置
# keys的使用方式同loggers

[handlers]
keys=hand01,hand02,hand03,hand04,hand05,hand06,hand07,hand08,hand09

# "formatters" 模块包含所有的 formatter 配置
# keys的使用方式同loggers

[formatters]
keys=form01,form02,form03,form04,form05,form06,form07,form08,form09

[logger_root]
level=NOTSET # DEBUG, INFO, WARN, ERROR, CRITICAL 或 NOTSET
# NOTSET: 如果 propagate=1, 输出参考 parent, 否则, 全部输出
handlers=hand01
qualname=(root) # root比较特殊, root需要加(), 别的不需要
propagate=1 # 此参数为非root logger使用
channel=
parent=

[logger_log02]
level=DEBUG
propagate=1
qualname=log02 # qualname 填写全称
handlers=hand02 # 使用的 handler 名称
channel=log02 # channel表示最末端部分名称
parent=(root) # 父logger为root

[logger_log03]
level=INFO
propagate=1 # 如果 propagate=1, 输出参考 parent
qualname=log02.log03 # 由于其 parent 为 log02, 所以全称为 log02.log03
handlers=hand03
channel=log03 # 填写最末名称, log02.log03 最后为 log03
parent=log02

[logger_log04]
level=WARN
propagate=0
qualname=log02.log03.log04
handlers=hand04
channel=log04
parent=log03

[logger_log05]
level=ERROR
propagate=1
qualname=log02.log03.log04.log05
handlers=hand05
channel=log05
parent=log04

[logger_log06]
level=CRITICAL
propagate=1
qualname=log02.log03.log04.log05.log06
handlers=hand06
channel=log06
parent=log05

[logger_log07]
level=WARN
propagate=1
qualname=log02.log03.log04.log05.log06.log07
handlers=hand07
channel=log07
parent=log06

[handler_hand01]
class=StreamHandler # 实例发送错误到流(类似文件的对象)
level=NOTSET # NOTSET 表示 level 参考 parent
formatter=form01
args=(sys.stdout,) # args 为传给 handler 构造函数的参数
stream=sys.stdout

[handler_hand02]
class=FileHandler # 实例发送错误到磁盘文件
level=DEBUG
formatter=form02
args=('python.log', 'w')
filename=python.log
mode=w

[handler_hand03]
class=handlers.SocketHandler # 实例发送日志到TCP/IP socket
level=INFO
formatter=form03
args=('localhost', handlers.DEFAULT_TCP_LOGGING_PORT)
host=localhost
port=DEFAULT_TCP_LOGGING_PORT

[handler_hand04]
class=handlers.DatagramHandler # 实例发送错误信息通过UDP协议
level=WARN
formatter=form04
args=('localhost', handlers.DEFAULT_UDP_LOGGING_PORT)
host=localhost
port=DEFAULT_UDP_LOGGING_PORT

[handler_hand05]
class=handlers.SysLogHandler # 实例发送日志到UNIX syslog服务，并支持远程syslog服务
level=ERROR
formatter=form05
args=(('localhost', handlers.SYSLOG_UDP_PORT), handlers.SysLogHandler.LOG_USER)
host=localhost
port=SYSLOG_UDP_PORT
facility=LOG_USER

[handler_hand06]
class=NTEventLogHandler # 实例发送日志到WindowsNT/2000/XP事件日志
level=CRITICAL
formatter=form06
args=('Python Application', '', 'Application')
appname=Python Application
dllname=
logtype=Application

[handler_hand07]
class=SMTPHandler # 实例发送错误信息到特定的email地址
level=WARN
formatter=form07
args=('localhost', 'from@abc', ['user1@abc', 'user2@xyz'], 'Logger Subject')
host=localhost
port=25
from=from@abc
to=user1@abc,user2@xyz
subject=Logger Subject

[handler_hand08]
class=MemoryHandler # 实例发送日志到内存中的缓冲区，并在达到特定条件时清空
level=NOTSET
formatter=form08
target=
args=(10, ERROR)
capacity=10
flushlevel=ERROR

[handler_hand09]
class=HTTPHandler # 实例发送错误信息到HTTP服务器，通过GET或POST方法
level=NOTSET
formatter=form09
args=('localhost:9022', '/log', 'GET')
host=localhost
port=9022
url=/log
method=GET

[handler_hand010]
# BaseRotatingHandler是所有轮徇日志的基类，不能直接使用。
# 但是可以使用RotatingFileHandler和TimeRotatingFileHandler。
class=handlers.RotatingFileHandler # 实例发送信息到磁盘文件，并且限制最大的日志文件大小，并适时轮徇
level=INFO
formatter=form02
args=('test.log', 'a', 10*1024*1024, 5)

###############################################################################
[formatter_form01]
format=F1 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form02]
format=F2 %(asctime)s %(pathname)s(%(lineno)d): %(levelname)s %(message)s
datefmt=

[formatter_form03]
format=F3 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form04]
format=%(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form05]
format=F5 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form06]
format=F6 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form07]
format=F7 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form08]
format=F8 %(asctime)s %(levelname)s %(message)s
datefmt=

[formatter_form09]
format=F9 %(asctime)s %(levelname)s %(message)s
datefmt=

# --- end of logconf.ini ----------------------------------------------------
logging在低版本（如 2.3.4）中用法有少许差别
def initial_logger():
    import logging
    LOG_FILENAME = 'test.log'
    logger = logging.getLogger()
    handler = logging.FileHandler(LOG_FILENAME)
    formatter = logging.Formatter('[%(asctime)s] %(filename)s[line:%(lineno)d] [%(levelname)s] %(message)s', '%Y-%m-%d %H:%M:%S')
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.NOTSET)
    return logger

def main():
    logger = initial_logger()
    logger.debug("test debug")

if __name__ == '__main__':
    main()
```
