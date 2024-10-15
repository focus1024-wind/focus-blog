---
title: flask请求时间记录和日志处理
description: flask请求时间记录和日志处理
date: 2024-02-21
slug: python/flask_with_logging
image: 
categories:
    - Python
tags:
    - Python
    - Flask
    - log
---

# 时间记录
在Python中，如果需要记录一个函数执行的时间，可以通过装饰器的方式来实现，避免在每个函数中进行重复编码。
```python
def time_summary(func):
    def wrapper(*args, **kwargs):
        # 以秒为单位的时间戳
        start = time.time()
        result = func(*args, **kwargs)
        logger.info(f"function use time {(time.time() - start) * 1000:.5f}ms")
        return result

    return wrapper

@time_summary
def hello_world():
    return "hello world"
```

# flask日志处理
在logging日志处理中，每次日志输出都是一个全新的`LogRecord`事件，所以每次的输出之间是相互不关联的。但是在实际的项目中，我们希望能够在日志中获取每一次请求所对应的输出时间。

为实现上述需求，我们需要将同一次请求的事件关联起来，添加相同的标识打印到日志中。在日志的格式化输出中，通过`LogRecord`属性打印标识，但是每次日志输出都是一个全新的`LogRecord`事件。flask中是存在上下文的，所以考虑在一次请求的上下文中添加指定标识，然后在每次输出日志时，`LogRecord`获取该标识进行输出，管理同一请求。

## 全局上下文实现
```python
import logging
import time
import uuid

from flask import Flask, g, has_app_context

app = Flask(__name__)


class RequestFilter(logging.Filter):
    def filter(self, record):
        if has_app_context() and hasattr(g, "trace_id"):
            record.trace_id = g.trace_id
        else:
            record.trace_id = None
        return record


handler = logging.StreamHandler()
handler.setFormatter(
    logging.Formatter(
        '[%(asctime)s] - [%(trace_id)s] - [%(levelname)s] - [%(message)s]')
)
handler.addFilter(RequestFilter())

logger = logging.getLogger()
logger.addHandler(handler)
logger.setLevel(logging.INFO)


def time_summary(func):
    def wrapper(*args, **kwargs):
        # 以秒为单位的时间戳
        start = time.time()
        result = func(*args, **kwargs)
        logger.info(f"function use time {(time.time() - start) * 1000:.5f}ms")
        return result

    return wrapper


@app.before_request
def add_trace_id():
    g.trace_id = str(uuid.uuid4())


@app.route("/", methods=["GET"])
@time_summary
def hello_world():
    logger.info("hello world")
    return "hello world"
# [2024-02-20 18:21:14,605] - [60b03671-10e4-4143-922e-2c336ff6df44] - [INFO] - [hello world]
# [2024-02-20 18:21:14,606] - [60b03671-10e4-4143-922e-2c336ff6df44] - [INFO] - [function use time 0.16093ms]
```
在上述示例中，通过`@app.before_request`拦截器在请求前向全局上下文中添加了`trace_id`属性，通过`Filter`在每次输出日志时获取请求全局上下文的`trace_id`属性，使每一次日志输出的请求通过uuid进行关联。
## 实现优化
随着微服务的发展，我们希望在多个微服务之间能够关联相同的uuid，但是通过全局上下文的方式只能保证每一次请求的uuid相同，原因在于我们的uuid是直接生成的，若是能在多个服务之间关联起uuid，当请求时，则可根据关联的uuid获取相同的日志内容。

考虑添加参数，但存在参数值时，以该值为同一请求日志追踪标识，不存在则生成。

这里在方法的请求头中添加参数实现，若在每个flask请求上添加参数实现，需要多次重复实现且当实现变更时，容易漏改
```python
from wsgiref.headers import Headers

@app.before_request
def add_trace_id():
    if request.headers.get("X-Trace-Id") is None:
        headers = Headers()
        for key, value in request.headers.items():
            headers[key] = value
        headers["X-Trace-Id"] = str(uuid.uuid4())
        request.headers = headers
    g.trace_id = request.headers.get("X-Trace-Id")
```
> 在flask中，headers是不可变对象，所以不可以直接修改headers对象。
> 虽然headers是不可变对象，但是headers指针却不是固定的，所以通过获取原来的headers内容，修改指针的方式来解决不可变对象问题（headers不可拷贝，所以以迭代遍历的方式获取）