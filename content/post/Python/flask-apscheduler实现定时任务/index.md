---
title: flask-apscheduler实现定时任务
description: flask-apscheduler实现定时任务
date: 2023-07-11
slug: python/flask/flask-apscheduler
image: 
categories:
    - Python
tags:
    - Python
    - Cron
---

flask-apscheduler是一个支持apscheduler的flask插件，通过flask-apscheduler可以很方便的定义定时任务。

# 安装flask-apscheduler
```shell
pip install flask-apscheduler
```

# flask-apscheduler组件
apscheduler库包含有组件：
- triggers（触发器）
- job stores（作业存储）
- executors（执行者）
- schedulers（调度程序）

## triggers触发器
在apscheduler中将一个定时任务称为`job`作业，在设置`job`时，需要为其选择特点的触发器。
**触发器**：确定作业运行时计算日期/时间的逻辑。
在apscheduler中具有以下三种内置的触发器类型：
- date: 期望在某个时间点运行一次作业时使用该触发器
	- run_date: 运行作业的日期/时间,为空则使用当前时间
	- timezone: 时区
- interval: 期望以固定的时间间隔运行作业时使用该触发器
	- weeks: 间隔的周数
	- days: 间隔的天数
	- hours: 间隔的小时数
	- minutes: 间隔的分钟数
	- seconds: 间隔的秒数
	- start_date: 间隔计算的起点。若指定则成指定时间开始，否则以当前时间开始。
	- end_date: 最晚可能触发的日期/时间
	- timezone: 时区
	- jitter: 最多延迟几秒执行
- cron: 当想在特定时间定期运行作业时使用该触发器
	- year: 4位数年份
	- month: 月
	- day: 一个月中的某一天天
	- week: ISO周（1～31）
	- day_of_week: 工作日的数字或者名称（0～6或mon, tue, wed, thu, fri, sat, sun）
	- hour: 小时
	- minute: 分钟
	- second: 秒
	- start_date: 最早可能的触发时间。若指定则成指定时间开始，否则以当前时间开始。
	- end_date: 最晚可能触发的日期/时间
	- timezone: 时区
	- jitter: 最多延迟几秒执行

# flask-apscheduler的使用
## 参数配置
### 特定于flask-apscheduler的配置选项
```python
# 是否开启api查看定时任务的配置
# 设置为True后，可以通过SCHEDULER_API_PREFIX接口访问定时任务配置界面
SCHEDULER_API_ENABLED: bool (default: False)
# 设置访问查看定时任务配置的api接口
SCHEDULER_API_PREFIX: str (default: "/scheduler")
# 调度程序endpoint前缀
SCHEDULER_ENDPOINT_PREFIX: str (default: "scheduler.")
# 允许访问调度器的主机
SCHEDULER_ALLOWED_HOSTS: list (default: ["*"])
```
在测试定时任务时，建议将`SCHEDULER_API_ENABLED`设置为`True`，方便前端查看定时任务。
若开启api后访问接口后端报错`KeyError: 'JSONIFY_PRETTYPRINT_REGULAR'`，可在配置中添加布尔类型的`JSONIFY_PRETTYPRINT_REGULAR`，值设置为`True`或`False`都可以，设置为`True`仅会在前端界面以更美观的方式显示内容。

### 适用于apscheduler的配置选项
```python
# 持久化配置
SCHEDULER_JOBSTORES: dict
# 执行器，线程池配置
SCHEDULER_EXECUTORS: dict
# 作业默认值配置
SCHEDULER_JOB_DEFAULTS: dict
# 时区设置
SCHEDULER_TIMEZONE: dict
```

## 定时任务设置方法
### 配置项设置定时任务
flask-apscheduler的定时任务在配置项中设置，通过字典保存作业的相关信息，多个作业时，在列表中进行作业的存储。
```python
class SchedulerConfig:
	JSONIFY_PRETTYPRINT_REGULAR = True
    SCHEDULER_API_ENABLED = True
	JOBS = [
        {
            'id': 'job1',
			# func通过`模块名:方法名`的方式进行设置
            'func': 'run:add',
            'args': (1, 2),
            'trigger': 'interval',
            'seconds': 3
        }
    ]
```
配置项设置的优点是配置麻烦，不直观，但是方便对定时任务进行统一管理

### 装饰器设置定时任务
在装饰器中设置定时任务，简单方便。
```python
@scheduler.task('interval', id='job1', args=(1, 2), seconds=3)
def add(a, b):
	print(a + b)
```

## 完整示例与定时任务的启动
定时任务的执行需要先初始化一个执行程序，添加好配置和作业后，通过`start()`方法启动定时任务。

### 完整示例（以装饰器的方式为例）
```python
import time

from flask import Flask
from flask_apscheduler import APScheduler

app = Flask(__name__)

scheduler = APScheduler()

start = time.time()


@scheduler.task('interval', id='job1', seconds=3)
def print_interval():
    global start
    print(f"间隔时间：{round(time.time() - start)}")
    start = time.time()


class Config:
    JSONIFY_PRETTYPRINT_REGULAR = True
    SCHEDULER_API_ENABLED = True


app.config.from_object(Config)

scheduler.init_app(app)

print(scheduler.api_enabled)
scheduler.start()

if __name__ == '__main__':
    app.run()
```
示例的执行结果如下所示：
```python
间隔时间：3
间隔时间：3
间隔时间：3
...
```
可以看到每3秒进行了一次打印操作，完美的执行了期望的要求。

### api查看
访问`${SCHEDULER_API_PREFIX}/jobs`接口：
```json
[
  {
    "id": "job1",
    "name": "print_interval",
    "func": "__main__:print_interval",
    "args": [],
    "kwargs": {},
    "trigger": "interval",
    "start_date": "2023-07-11T12:09:49.057865+08:00",
    "seconds": 3,
    "misfire_grace_time": 1,
    "max_instances": 1,
    "next_run_time": "2023-07-11T12:11:31.057865+08:00"
  }
]
```