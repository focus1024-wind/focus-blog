---
title: Python日志（logging）
description: Python日志（logging）
date: 2024-02-21
slug: python/logging
image: 
categories:
    - Python
tags:
    - Python
    - Flask
    - log
---

# 日志
## 日志的概念
日志：记录系统运行状态、操作和事件的文件或数据记录。在计算机领域中，日志通常用于跟踪系统的运行情况、故障排查、性能分析和安全审计等方面。日志记录通常包括时间戳、事件描述、发生事件的位置等信息，可以帮助系统管理员或开发。
## 日志的作用
- 故障排查：日志记录了系统运行过程中的各种操作和事件，可以帮助开发人员快速定位和解决问题，提高系统的稳定性和可靠性。
- 性能监控：通过分析日志可以了解系统的性能表现，包括响应时间、吞吐量、资源利用率等指标，帮助优化系统性能。
- 安全审计：日志记录了系统的操作历史，可以用于追踪用户行为、检测异常操作、防止安全漏洞等，保障系统的安全性。
- 统计分析：日志中包含了大量的数据，可以通过分析日志来了解用户行为、业务趋势、用户偏好等信息，为业务决策提供数据支持。
- 运营监控：通过监控日志可以实时了解系统的运行状态，及时发现问题并采取措施，保障系统的正常运行。
## 日志等级
| 日志级别 | 使用场景 |
| ---- | ---- |
| DEBUG | 用于输出调试信息，通常用于开发和测试阶段，帮助开发人员定位问题和调试程序。 |
| INFO | 用于输出一般信息，表示程序正常运行的状态，可以用来跟踪程序的执行流程。 |
| WARN | 用于输出警告信息，表示程序遇到了一些不严重的问题或异常情况，但程序仍然可以继续运行。 |
| ERROR | 用于输出错误信息，表示程序遇到了严重的问题或异常情况，可能会导致程序崩溃或无法正常运行。 |
| FATAL | FATAL：用于输出致命错误信息，表示程序遇到了无法恢复的严重问题，程序可能会立即终止运行。 |
> 根据不同的日志框架，支持的日志等级会有细微差别

# Python logging
logging是Python官方的日志模块，用于记录程序运行时的日志信息。logging提供了灵活的日志记录功能，可以根据不同的需求配置不同的日志记录器、处理器和格式器。

由于logging的强大灵活，在Python中，日志模块logging得到了广泛的应用。

## logging主要组件
- Logger（日志记录器）：用于创建日志记录器对象，可以通过设置级别、添加处理器等来控制日志记录的行为。
- Handler（处理器）：用于将日志记录发送到不同的目的地，如控制台、文件、网络等。
- Formatter（格式器）：用于定义日志记录的格式，包括时间、级别、消息等内容的显示方式。

## logging 根记录器
### 默认输出
在导入`logging`模块后，可以在不实例logger的情况下，直接使用`logging`进行日志打印，`logging`模块将会在根记录器（root logger）上打印日志信息。
```python
import logging

logging.debug('This is a info message')
logging.info('This is a debug message')
logging.warning('This is a warning message')
logging.error('This is a error message')
logging.critical('This is a critical message')

# WARNING:root:This is a warning message
# ERROR:root:This is a error message
# CRITICAL:root:This is a critical message
```
如上所示，导入`logging`模块后，可直接在控制台打印出部分日志信息（logging默认级别为warning），通过查看`logging`模块源码可知，`logging`默认情况下是在根记录器（root logger）上通过根处理器（root handler）进行的日志输出。

`logging`源码debug：
```python
def debug(msg, *args, **kwargs):
    """
    Log a message with severity 'DEBUG' on the root logger. If the logger has
    no handlers, call basicConfig() to add a console handler with a pre-defined
    format.
    """
    if len(root.handlers) == 0:
        basicConfig()
    root.debug(msg, *args, **kwargs)
```
### 根记录器配置
`logging`的默认根记录器输出格式固定，日志级别固定，输出流固定。直接使用默认的根处理器，日志固定，直接使用默认输出，从功能上讲和`print()`打印信息并没有本质区别，为了更好的实现日志输出，通过阅读上面`debug()`方法可知，在root logger中，通过`basicConfig()`方法进行handler和输出格式的设置。

basicConfig默认创建一个`StreamHandler`，讲日志输出到标准输出流（控制台）中，使用`BASIC_FORMAT`格式字符串设置格式化程序，然后将处理程序添加到根记录器。basicConfig常用支持参数：
| 参数名称 | 参数描述 |
| ---- | ---- |
| filename | 将日志输出到指定的文件中，而不是控制台 |
| filemode | 在指定`filename`参数的情况下，指定打开文件格式，默认为`a` |
| format | 日志输出格式，仅支持以字符串的模式指定，不支持通过Formatter对象指定（只能使用内置的LogRecord属性），默认以`%`（可通过`style`属性指定）格式化内置LogRecord属性。如：在`format`参数中添加`%(levelname)`来在打印日志时输出日志级别 |
| datefmt | 日期/时间格式 |
| level | 设置日志级别 |
| stream | 使用指定的流初始化`StreamHandler`，与`filename`不兼容 |

如，以下示例，我们讲日志输出到指定文件中：
```python
import logging

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s %(filename)s %(levelname)s %(message)s',
                    datefmt='%a %d %b %Y %H:%M:%S',
                    filename='my.log',
                    filemode='w')

logging.debug('This is a info message')
logging.info('This is a debug message')
logging.warning('This is a warning message')
logging.error('This is a error message')
logging.critical('This is a critical message')

> 下面为`my.log`文件内容
Tue 20 Feb 2024 16:41:52 test.py INFO This is a debug message
Tue 20 Feb 2024 16:41:52 test.py WARNING This is a warning message
Tue 20 Feb 2024 16:41:52 test.py ERROR This is a error message
Tue 20 Feb 2024 16:41:52 test.py CRITICAL This is a critical message
```
## logging format
上面介绍了logging root logger的使用，在处理日志时，打印日志的格式也是重要的一个环节，在`logging`模块中，内置了许多`LogRecord`属性，可以在`Formatter`类中进行查看，其中`LogRecord`用来记录日志事件，`Formatter`对象用来讲`LogRecord`事件转换为格式化文本信息。
### 内置`LogRecord`属性
| 属性格式 | 描述 |
| ---- | ---- |
| %(name)s | 日志通道名称 |
| %(levelno)s | 日志级别对应的数字信息 |
| %(levelname)s | 日志级别对应的文字信息 |
| %(pathname)s | 日志记录的源文件的完整路径名 |
| %(filename)s | 日志记录文件名 |
| %(module)s | 日志文件名名称部分（不包含文件后缀） |
| %(lineno)d | 日志行号 |
| %(funcName)s | 日志所在方法名称 |
| %(created)f | 记录`LogRecord`的事件的时间，`time.time()`格式 |
| %(asctime)s | 记录`LogRecord`的事件的文本时间 |
| %(msecs)d | 记录`LogRecord`的事件时间 |
| %(relativeCreated)d | 记录`LogRecord`的事件相对于加载`logging`模块时间，单位毫秒 |
| %(thread)d | 线程ID |
| %(threadName)s | 线程名称 |
| %(process)d | 进程ID |
| %(message)s | 日志内容信息 |
### 添加`LogRecord`属性
默认的`LogRecord`属性能满足我们日志的大部分格式化需求，但是针对某些特定场景，我们需要额外的属性来进行格式化输出。

在`logginer`模块中，通过`Filter`实例对`LogRecord`对象进行过滤，所以我们可以通过`Filter`来向`LogRecord`中添加定制化属性。通过重写`Filter`的`filter(self, record)`方法以实现添加属性。
```python
import logging

class CustomFilter(logging.Filter):
    def filter(self, record):
        record.trace_id = "trace_id"
        return record

handler = logging.StreamHandler()
handler.setFormatter(
    logging.Formatter('[%(asctime)s] - [%(trace_id)s] - [%(levelname)s] - [%(message)s]')
)
handler.addFilter(CustomFilter())

logger = logging.getLogger()
logger.addHandler(handler)
logger.setLevel(logging.INFO)

logger.info("Add field in LogRecord")

# [2024-02-20 17:17:38,280] - [trace_id] - [INFO] - [Add field in LogRecord]
```
参考如上方法编写`Filter`类，添加定制化属性。

## logging handler
在`logging`模块中，通过handler对象将日志输出的指定目的地。

### logging默认handler
#### NullHandler空操作handler
空操作handler，不进行任何处理，无参数。
#### StreamHandler流handler
将日志信息输出到指定流中，默认情况下输出到标准输出流中。
```python
class logging.StreamHandler(stream=None)
```
#### FileHandler文件handler
继承自`StreamHandler`（文件本身也是一种流），讲日志输出到指定文件中。
```python
class logging.FileHandler(filename, mode='a', encoding=None, delay=False, errors=None)
```
- delay：delay为True时，文件只有被写入日志时才会被打开。即默认情况下，直接打开文件，delay为True时，在写入时打开文件。
- errors：`FileHandler`通过Python默认方法`open()`打开文件，errors为编解码报错的处理模式
### logging扩展handler
以上默认handler位于`logging`模块下，`logging`模块还提供一些更高级的处理器位于`logging.handlers`模块下。
#### logging.handlers.RotatingFileHandler循环大小日志文件处理
用于记录日志信息到一组文件的处理程序，当当前文件达到一定大小时，该处理程序会从一个文件切换到下一个文件。
```python
class logging.handlers.RotatingFileHandler(filename, mode='a', maxBytes=0, backupCount=0, encoding=None, delay=False, errors=None)
```
- maxBytes：文件的最大字节数，当文件长度接近maxBytes将会发生滚动
- backupCount：扩展文件个数，如当backupCount为2，filename为root.log时，当日志达到maxBytes，将会依次创建root.log.1，root.log.2。日志将会依次在root.log，root.log.1，root.log.2文件中循环输出，覆盖之前的内容
#### logging.handlers.TimedRotatingFileHandler循环时间日志文件处理
用于记录日志信息到一组文件的处理程序，在指定时间间隔进行日志文件组的循环。
```python
class logging.handlers.TimedRotatingFileHandler(filename, when='h', interval=1, backupCount=0, encoding=None, delay=False, utc=False, atTime=None, errors=None)
```
- when：决定时间间隔的类型	
- interval：决定多少的时间间隔
- backupCount：决定保存日志文件数量。超过数量就会丢弃覆盖老的日志文件。
#### logging.handlers.WatchedFileHandler监控日志文件状态处理
用于监视文件的状态，如果文件被改变了，那么就关闭当前流，重新打开文件，创建一个新的流。通过使用newsyslog和logrotate等程序执行日志文件旋转，该handler仅支持linux/unix，不支持Windows。
```python
class logging.handlers.WatchedFileHandler(filename, mode='a', encoding=None, delay=False, errors=None)
```
## logger日志记录器输出日志
上面我们通过`logging`模块直接输出了日志信息，实际上是通过默认根记录器进行的日志处理和输出。在正式的环境中，更推荐采用自定义的logger对象进行日志输出。

一般logger流程：
- 声明handler
- 设置handler的日志格式
- 获取logger对象
- 添加handler和日志级别

```python
import logging

handler = logging.StreamHandler()
handler.setFormatter(
    logging.Formatter('[%(asctime)s] - [%(levelname)s] - [%(message)s]')
)

logger = logging.getLogger()
logger.addHandler(handler)
logger.setLevel(logging.INFO)
```