---
title: Python自动化提交每日任务
description: Python自动化提交每日任务
date: 2024-01-26
slug: python/auto_daily_task_with_flask
image: 
categories:
    - Python
tags:
    - Python
    - Flask
---

> 自动化任务根据具体的环境不同，任务的调用执行方式会有不同，要根据具体任务具体分析
> 本博文仅就博主自己的自动化每日任务进行分析

最近小组根据每日计划的填报情况来进行自动化打分，但是有时总会忘记填写计划导致会扣分，所以为避免此类情况，考虑编写自动化脚本进行任务填写。

对自动化每日任务进行分析，在自动化提交每日任务中需要涉及一下任务:
- 接口的自动鉴权
- 接口数据解析
- 工作日的判断
- 每日任务的自动化生成
- 定时执行
- 自动任务失败的消息发送

在实现自动化提交每日任务中，重点在于对于**每日任务的自动提交**，**任务的定时执行**。Python中可以通过`schedule`或者`time.sleep()`来协助我们执行定时任务，但这里问了实现更强大的功能，考虑采用`flask`框架进行开发，通过`flask-APScheduler`来实现定时任务的执行和失败消息的推送。

# 日志配置
新建`conf.py`文件，添加如下配置:
```python
import logging

logging.basicConfig(
    format='[%(asctime)s] %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

```
可根据自己需要决定是否添加日志信息

# 接口自动鉴权和计划提交
前后端的登陆一般通过Cookie、Session、Token等方式进行登陆，通过开发者工具查看请求头信息，可以根据请求头信息等内容获取系统的具体登陆鉴权方式。
- cookie: 由服务器生成后发送给客户端浏览器。浏览器将cookie以kv形式存储在客户端，在请求时同时携带cookie访问服务器。
- session: 在cookie的存储中，主要保存在客户端。在sesson则是服务器记录客户端状态的一种机制。
	- 服务端给每个客户端分配不同的“标识”，在客户端请求服务器时会携带该“标识”（浏览器中一般采用cookie的方式）
	- session相比于cookie，在服务器端中也存储了用户信息，相比于单纯的cookie更安全，但是由于将session储存在服务器端，同时也破坏了服务器的无状态性，当服务器端采用集群进行负载均衡时，由于session不会共享，会在浏览器端出现随机性的不可访问现象
- token: token是一种无状态的由服务器签发的访问凭证，用户身份校验成功后，由服务器签发token给客户端，客户端在访问服务器时，携带token进行访问。

## token与cookie的比较
| | cookie | token |
| ---- | ---- | ---- |
| 定义 | cookie是HTTP规范的一部分 | token是客户服务端自定义的一种数据交换方式 |
| 存储 | cookie存储在浏览器中 | token可以根据客户端需要决定存储位置 |
| 安全性 | cookie不加密，可以被分析和拦截 | token是无状态的，token泄漏后，同样也会影响到数据的安全访问。根据token的编码方式同样会造成一定程度上的信息泄漏。已JWT（JSON Web Token）为例，已知JWT可以直接解析后header、payload内容，JWT根据header、payload和服务端密钥生成签名部分，通过签名可以防止token被篡改 |
| 扩展性 | cookie一般同时和session使用，服务端需要存储会话信息，会影响服务端端负载均衡扩展 | token是无状态的，一次签发，直到其过期前，针对一致的签名方式都是有效的，可以在集群内无状态访问 |
| 跨域支持 | cookie受同源请求限制，无法在跨域时自动发送cookie | token可以在跨域时携带信息，进行传输 |
| 时效性 | cookie会随着浏览器的关闭而销毁 | token签发后可一直保存，根据签发设置过期时间，除非token过期或者服务器签名方式变更，以签发的token可一直携带鉴权信息 |

## session(cookie)鉴权实现
在session中，由服务端存储session值，同时在客户端存储cookie值，在用户登陆时，保存session信息，生成cookie信息返回给前端。

在客户端访问浏览器时，通过请求头携带cookie信息进行访问。cookie信息一般是以kv形式存储在客户端。
```python
from datetime import datetime
from urllib import parse

import requests

from auto_task.auto_task.conf import logger


def login(username, password) -> str:
    """
    自动登陆
    获取登陆的session值
    :param username:
    :param password:
    :return: session
    """
    url = f"${login_url}?username={username}&password={password}"

    response = requests.get(url)

    session = response.cookies.values()[0]
    return session


def submit_daily_task(username, password, content, today=None) -> (bool, str):
    """
    提交每日任务
    :param username:
    :param password:
    :param content: 每日任务内容
    :param today: 每日任务日期
    :return:
    """
    if today is None:
        today = datetime.now()

    url = "${task_url}/routine/daily"

    session = login(username, password)

    headers = {
		# 通过请求头设置cookie信息
        'Cookie': f'session={session}',
        'Accept': '*/*',
		# 根据具体访问方式设置
        'Content-Type': 'application/x-www-form-urlencoded'
    }

    payload = {
        "date": today,
        "content": content
    }

    logger.info(f"{username} {today}计划:\n{payload}")

	# x-www-form-urlencoded类型编码
    payload = parse.urlencode(payload)

    response = requests.request("POST", url, headers=headers, data=payload)

	# unicode cjk编码
    result = response.text.encode("utf-8").decode("unicode-escape")

    if '"修改成功"\n' == result:
        logger.info(f"{username} {today} 计划提交成功")
        return True, result
    else:
        logger.error(f"{username} {today} 计划提交失败")
        logger.error(f"{username} {today} 失败信息: {result}")
        return False, result
```

### urlencode（url编码）
由于在我的任务中，提交计划是通过`application/x-www-form-urlencoded`方式请求，所以对于数据信息需要通过url编码
```python
from urllib import parse

payload = {
    "date": today,
    "content": content
}
payload = parse.urlencode(payload)
```

### unicode编码
在一些系统中，返回内容中中文字符输出显示为`\uxxxx\uxxxx...`的形式。在一些系统中，针对CJK(中日韩统一表意文字)会返回其unicode内容，对这些内容，为了更好对访问，应把其编码为正常显示内容。实际上，在返回显示为`\uxxx`的内容上，在内容中是被编码为`\\uxxxx`。在Python中，通过`unicode-escape`来处理unicode编解码问题。
```python
result = response.text.encode("utf-8").decode("unicode-escape")
```

## token(JWT)鉴权实现
在token传参中，一般在请求头中通过`Autherization: Bearer ${token}`进行token的传参鉴权
```python
from datetime import datetime
from urllib import parse

import requests

from auto_task.auto_task.conf import logger


def submit_daily_task(username, password, content, today=None) -> (bool, str):
    """
    提交每日任务
    :param username:
    :param password:
    :param content:
    :param today:
    :return:
    """
    if today is None:
        today = datetime.now()

    url = "http://uncleyiba.com:1016/routine/daily"

    session = login(username, password)

    headers = {
        'Authorization': f'Bearer {session}',
        'Accept': '*/*',
        'Content-Type': 'application/x-www-form-urlencoded'
    }

    payload = {
        "date": today,
        "content": content
    }

    logger.info(f"{username} {today}计划:\n{payload}")

    payload = parse.urlencode(payload)

    response = requests.request("POST", url, headers=headers, data=payload)

    result = response.text.encode("utf-8").decode("unicode-escape")

    if '"修改成功"\n' == result:
        logger.info(f"{username} {today} 计划提交成功")
        return True, result
    else:
        logger.error(f"{username} {today} 计划提交失败")
        logger.error(f"{username} {today} 失败信息: {result}")
        return False, result
```
在鉴权时主要通过修改请求头来修改传参鉴权方式。

# HTML解析
在一些非前后端分离的项目中。通过接口请求，会返回前端HTML内容，HTML内容以str字符串的信息保存，直接对字符串操作获取标签内容麻烦易出错，此时可以通过`bs4`库进行HTML内容的解析。

安装`bs4`：
```shell
pip install bs4
```

**HTML解析**:
```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(response.text, 'html.parser')
textarea_text = soup.select("${标签名称}")[0].get_text()
```

# 每日任务的自动化生成
我们组内部，每日计划主要为包括昨日计划和今日计划两部分内容，方便对昨日工作进行总结和今日内容进行展望。所以自动化内容中，可以通过获取昨日的工作和预制工作内容生成今日自动化提交的工作任务。

由于我们组内部获取获取历计划内容前后端未分离，所以通过上面的HTML解析获取历史计划，对历史计划按照指定的方式进行撰写，如以`\n\n`进行昨日任务和今日任务的分割，这样即可更好的在自动化任务中处理自己的历史计划。
```python
def get_last_weekday_content(username, password, last_date=None) -> str:
    """
    获取上一次工作日提交内容
    根据\n\n进行内容分割
    :param username:
    :param password:
    :param last_date:
    :return:
    """
    if last_date is None:
        last_date = date.today() - timedelta(days=1)

    url = f"${last_weekday_content}&start_date={last_date}&end_date={last_date}"

    session = login(username, password)

    headers = {
        'Cookie': f'session={session}',
        'Accept': '*/*',
        'Connection': 'keep-alive'
    }

    response = requests.get(url, headers=headers)

    soup = BeautifulSoup(response.text, 'html.parser')
    textarea_text = soup.select("textarea")[0].get_text()
    last_date_content = textarea_text.split(r"\n\n")[0]
    return last_date_content.replace("\\n", "\n").encode("utf-8").decode("unicode-escape")
```

# 判断是否为工作日
python的第三方库`holidays`中可以进行判断是否为中国法定节假日信息，也可以通过爬虫相关技术获取节假日信息（不推荐，需要连接公网，且依赖第三方系统）。

由于我们组内部可以看到全组成员的计划，且不同公司机构内部的节假日可能存在不同，这里考虑不采用第三方判断是否为法定节假日信息进行工作日判断，而是由组内成员自动判断。

考虑获取全组的今日工作内容，若今日已填写的工作计划大于指定人数，判断今日为工作日，进行自动化填写。
```python
def judge_weekday(username, password, today=None, user_threshold: int = 5) -> bool:
    """
    判断是否为工作日，是否需要提交每日任务
    根据已提交每日计划人数进行判断
    :param username:
    :param password:
    :param today:
    :param user_threshold:
    :return:
    """
    if today is None:
        today = datetime.now().strftime("%Y-%m-%d")

    url = f"http://uncleyiba.com:1016/routine/daily?mode=1&date={today}"

    session = login(username, password)

    headers = {
        'Cookie': f'session={session}',
        'Accept': '*/*',
        'Host': 'uncleyiba.com:1016',
        'Connection': 'keep-alive'
    }

    response = requests.get(url, headers=headers)

    soup = BeautifulSoup(response.text, 'html.parser')
    textarea_list = [
        textarea.get_text().encode("utf-8").decode("unicode-escape")
        for textarea in soup.select("textarea")
        if len(textarea.get_text().encode("utf-8").decode("unicode-escape")) > 15
    ]
    # 带一个自己的昨日工作计划
    if len(textarea_list) > (user_threshold + 1):
        return True
    else:
        return False
```

# 自动任务失败，消息通知
从成本角度，这里考虑采用邮件进行任务推送。采用`flask-mail`库发送邮件

**依赖库安装：**
```shell
pip install flask flask-mail
```

## SMTP邮件参数配置
若是采用自动发送邮件，需要在邮箱中开启SMTP服务。以QQ邮箱为列，在设置 > 账号 > POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务 > 服务状态 中开启服务。

按照服务步骤开启服务后，会生成一串字符串，请妥善保管和该内容，不要泄露。

在采用邮件服务中，一般需要配置邮件服务器地址、端口（SMTP协议默认端口为465）、用户名（你的邮箱号）、密码（开启服务后生成的一串字符串）、协议（TLS或SSL协议）。

在flask-mail中，以如下方式添加邮件服务配置:
```python
import os

from flask import Flask
from flask_mail import Mail


class Config:
	# 请填写或环境变量配置自己的邮箱服务器内容
    MAIL_SERVER = os.getenv("MAIL_SERVER", "")
    MAIL_PORT = os.getenv("MAIL_PORT", 465)
    MAIL_USERNAME = os.getenv("MAIL_USERNAME", "")
    MAIL_PASSWORD = os.getenv("MAIL_PASSWORD", "")
	# 这里采用SSL加密协议，可以根据需要选择
    MAIL_USE_TLS = os.getenv("MAIL_USE_TLS", False)
    MAIL_USE_SSL = os.getenv("MAIL_USE_SSL", True)


app = Flask(__name__)

# 从类对象中获取flask参数配置
app.config.from_object(Config)

mail = Mail(app)
```

## 发送邮件
在flask-mail中，通过`flask_mail.Message`类来封装要发送的邮件信息，通过`flask_mail.Mail`对象来发送邮件到指定邮箱中。

在Message类中：
- subject: 邮件主题
- sender: 邮件发送者，一般为配置的MAIL_USERNAME
- recipients: 收件人列表，注意类型为列表,
- body: 邮件内容
- attachments: 附件内容

在flask-mail中，发送邮件需要在flask的上下文中，在`with app.app_context()`上下文中执行发件操作。

```python
# 这里由于自动任务收件人一般比较固定，将收件人固定在Config类中
class Config:
	# 获取环境变量的MAIL_RECIPIENT_USER参数或默认收件人参数。
	# 若存在，通过,分割多个收件人，生成收件人列表
	# 若不存在，将默认发件人设置为收件人
	# 由于环境变量只能为字符串，所以手动设置格式进行序列化
    MAIL_RECIPIENT_USER = os.getenv("MAIL_RECIPIENT_USER").replace(" ", "").split(",") \
        if os.getenv("MAIL_RECIPIENT_USER") is not None \
        else [MAIL_USERNAME]

def send_mail(username, message):
    logger.error(f"{username} 填报失败，发送错误邮箱")
    with app.app_context():
        msg = Message(
            subject=f"{date.today()}工作计划填报",
            sender=Config.MAIL_USERNAME,
            recipients=Config.MAIL_RECIPIENT_USER,
            body=f"{date.today()}工作计划填报失败\n错误信息为{message}"
        )
        mail.send(msg)
```

# 完整任务与定时执行
根据以上内容，我们将自动化提交每日任务中要处理的事务都已经原子化的准备好了，然后对以上事务进行汇总，使其能够在每次执行时，完整的提交每日计划。

## 完整任务
在进行操作时，需要账号密码和预定义的提交内容。考虑到可能存在多人同时使用自动化任务的情况，为更好的配置，这里采用`${username}:${password}:${content}@${username}:${password}:${content}@...`的形式进行配置。

首先，序列化好用户名、密码、预定义提交内容后，判断是否为工作日，若是，获取该用户的上一次工作任务，根据预定义内容进行生成今日工作内容。进行任务提交，若提交失败，邮件告知。
```python
def auto_daily_task():
    tasks = os.getenv("TASK", "daichaoxiong:123:今日计划")
    tasks = tasks.split("@")
    for task in tasks:
        username = task.split(":")[0]
        password = task.split(":")[1]
        content = task.split(":")[2]

        logger.info(f"{username} 开启执行自动化任务")

        is_weekday = judge_weekday(username, password)
        if is_weekday:
            logger.info(f"{username} 是工作日，进行计划填报")

            # 工作日，提交计划
            today = date.today()

            # 获取上一次工作日提交内容
            last_weekday = today - timedelta(days=1)
            last_weekday_content = None
            while last_weekday_content is None or "" == last_weekday_content:
                last_weekday_content = get_last_weekday_content(username, password, last_weekday)
                last_weekday = last_weekday - timedelta(days=1)

            # 生成今日要提交内容
            content = f'{str(today).replace("-", ".")}\n{content}\n\n{last_weekday_content}'

            # 任务提交
            is_submit, message = submit_daily_task(username, password, content, today)

            if not is_submit and os.getenv("USE_MAIL", True):
                send_mail(username, message)
        else:
            break
```

## 定时执行
任务设置好后，执行该任务虽然确实能进行每日任务的提交，但是若是每次提交都要执行代码，那就失去了自动化的意义。这里通过`Flask-APScheduler`设定于每日的指定时间进行执行。

`Flask-APScheduler`的使用可以参考[flask-apscheduler实现定时任务](http://focus1024.com/post/python/flask/flask-apscheduler/)内容。

```python
import os

from flask import Flask
from flask_apscheduler import APScheduler


class Config:
    JSONIFY_PRETTYPRINT_REGULAR = True
    SCHEDULER_API_ENABLED = True


app = Flask(__name__)

scheduler = APScheduler()

app.config.from_object(Config)

scheduler.init_app(app)
scheduler.start()


@scheduler.task('cron', id='auto_daily_task', hour=os.getenv("CRON_HOUR", "10"), minute=os.getenv("CRON_MINUTE", "45"))
def auto_daily_task():
    ...
```

将以上代码汇总后，一个完全的自动化提交每日任务的任务就已经实现了。针对该任务目前有以下改进点，未完待续：
- 一个人的任务失败时会将失败信息发送给配置邮箱的所有用户，可考虑将用户和其邮箱配置一一指定，失败信息只发送给本人
- 条件满足的情况下，自动化任务定时定点执行，若此时用户已经填写了每日工作，则用户的每日工作会被不完整的预定义工作内容替代，应判断用户是否已提交今日工作在进行填报
- 是否填报的依赖于已填报人数，若有工作日的已填报人数少于预制阈值或非工作日加班人数大于预制阈值，则不能很好的判断