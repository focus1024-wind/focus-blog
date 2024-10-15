---
title: Python操作RabbitMq
description: Python操作RabbitMq
date: 2023-07-14
slug: python/python_with_rabbitmq
image: 
categories:
    - Python
    - MQ
tags:
    - Python
    - MQ
    - RabbitMq
---

# Docker部署RabbitMq
```yaml
version: '3.8'
services:
  rabbitmq:
    image: rabbitmq:3.11.19
    container_name: rabbitmq
    hostname: rabbitmq
    ports:
	  # amqp协议通讯端口（对外服务必开）
      - 5672:5672
	  # RabbitMq自带管理界面访问端口
      - 15672:15672
    environment:
      - RABBITMQ_DEFAULT_VHOST=${vhost}
      - RABBITMQ_DEFAULT_USER=${user}
      - RABBITMQ_DEFAULT_PASS=${password}
```
RabbitMq自带有专门的管理界面，可以在其管理界面对RabbitMq进行管理查看等操作。
RabbitMq的管理界面的对外端口为`15672`，当我们启动RabbitMq后，需要启动管理界面插件后才能访问界面。
```shell
# 进入容器内部
docker exec -it rabbitmq bash
# 启动插件，启动管理界面
rabbitmq-plugins enable rabbitmq_management
```

# Python操作RabbitMq
## 安装依赖库`pika`
```shell
pip install pika
```

## 获取RabbitMq连接，基本配置
```python
# 连接RabbitMq
connect = pika.BlockingConnection(pika.URLParameters("amqp://${user}:${password}@${ip}:${port}/${vhost}"))
# 创建通讯信道
channel = connect.channel()
# 创建（声明）消息队列，该方法在没有队列时会主动创建队列
channel.queue_declare(queue=${queue_name})

# 关闭RabbitMq连接
channel.close()
connect.close()
```

## 发布者模式（发送数据）
```python
channel.basic_publish(exchange="", routing_key=${queue_name}, body=f"你好 World!".encode(encoding="UTF-8"))
```
- exchange: 在RabbitMq中，发布者不会直接将信息推送到队列中，而是而是先将消息投递到exchange中，在由exchange转发到具体的队列，队列再将消息以推送或者拉取方式给消费者进行消费。消息队列通过在在消费者和生产者中引入了exchange的概念，当压力增长的情况下，可以通过配置exchange在不停机的情况下调整系统资源，缓解服务压力
	- exchange为空表示设置为简单exchange模式
- routing_key: 传送到的队列名称
- body: 消息内容，在RabbitMq中，通过流的方式传输信息，所以需要对传输内容进行编码

### 发布者模式完整示例
```python
import pika

connect = pika.BlockingConnection(pika.URLParameters("amqp://${user}:${password}@${ip}:${port}/${vhost}"))
channel = connect.channel()
channel.queue_declare(queue=${queue_name})

channel.basic_publish(exchange="", routing_key=${queue_name}, body=f"你好 World!".encode(encoding="UTF-8"))

channel.close()
connect.close()
```

## 消费者模式（接收数据）
```python
def callback(channel, method, properties, body: bytes):
	"""
	自定义回调函数
	:param channel: BlockingChannel
	:param method: spec.Basic.Deliver
	:param properties: spec.BasicProperties
	:param body: bytes
	"""
	print(f"消费者接收数据: {body.decode('UTF-8')}")

# 绑定消息队列，接收数据
channel.basic_consume(queue=${queue_name}, auto_ack=True, on_message_callback=callback)

# 阻塞函数，启动后会一直运行等待发布者发送数据
channel.start_consuming()
```
- auto_ack: 自动确定配置，通过ACK机制（消息确认机制）确认消息是否被正确接收，确保消息的正确传输，不会丢失

### 消费者模式完整示例
```python
import pika

connect = pika.BlockingConnection(pika.URLParameters("amqp://${user}:${password}@${ip}:${port}/${vhost}"))
channel = connect.channel()
channel.queue_declare(queue=${queue_name})

def callback(channel, method, properties, body: bytes):
	"""
	自定义回调函数
	:param channel: BlockingChannel
	:param method: spec.Basic.Deliver
	:param properties: spec.BasicProperties
	:param body: bytes
	"""
	print(f"消费者接收数据: {body.decode('UTF-8')}")

# 绑定消息队列，接收数据
channel.basic_consume(queue=${queue_name}, auto_ack=True, on_message_callback=callback)

# 阻塞函数，启动后会一直运行等待发布者发送数据
channel.start_consuming()
```
- 设置为交换模式的情况下，一个发布者对应一个消费者。当在RabbitMq中注册多个消费者时，消息队列中的数据将会以轮训的方式发送到消费者中。