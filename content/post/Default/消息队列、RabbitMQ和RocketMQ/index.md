---
title: 消息队列、RabbitMQ和RocketMQ
description: 消息队列、RabbitMQ和RocketMQ
date: 2023-04-05
slug: mq/mq_rabbitmq_rocketmq
image: 
categories:
    - Default
tags:
    - MQ
---

# MQ（Message Queue）消息队列
## 什么是MQ
- MQ（Message Queue）消息队列，是基于数据结构中“先进先出”的一种数据结构。简单来说就是在消息传输过程中保存消息对容器。
- 一般用来解决应用解耦，异步消息，流量削峰等问题
- 实现高性能，高可用，可伸缩和最终一致性架构。
## 为什么使用消息队列
- 在高并发的场景下，由于来不及处理同步请求，请求会发生堵塞。通过消息队列，可以异步的处理请求，缓解系统压力
- 当系统资源有限时，不断的向系统发起请求，超过系统所能处理请求的最大阈值，可能会导致系统崩溃。
- 当发起请求需要立即获得回调信息，而请求的处理需要消费一点的时间时。如：发送邮件，系统下单，模型训练，远程接口调用等
## 大生产环境下，为什么选择用RabbitMQ之类的消息队列，而不是使用Redis这类可以做队列的NoSql数据库
- 把消息插入NoSql数据库，不如队列的入队出队操作简单
- 频繁的向数据库添加新的消息会增大数据库的负载
- 如果并发的处理数据库，需要对数据库进行上锁或处理会话等操作，甚至可能出现死锁等现象，而使用消息队列不需要考虑这些情况
- 当消息越来越多时，数据库需要定期对消息进行清理
- 消息队列在消费者和生产者中引入了exchange的概念，当压力增长的情况下，可以通过配置exchange在不停机的情况下调整系统资源，缓解服务压力
## MQ的实现方式
### AMQP
AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

在AMQP协议中，生产者将消息发送到Exchange中，通过Exchange将消息路由到不同的Queue中，由消费者从Queue中获取消息进行处理。由于Exchange的存在，在消息队列中可以通过配置Exchange将消息发送给不同的队列，缓解调整系统资源。

### JMS
JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。
JMS是一种与厂商无关的 API，用来访问收发系统消息，它类似于JDBC(Java Database Connectivity)。这里，JDBC 是可以用来访问许多不同关系数据库的 API，而 JMS 则提供同样与厂商无关的访问方法，以访问消息收发服务。JMS 使您能够通过消息收发服务（有时称为消息中介程序或路由器）从一个 JMS 客户机向另一个 JMS客户机发送消息。JMS消息通常有两种类型：（1）点对点（Point-to-Point）。在点对点的消息系统中，消息分发给一个单独的使用者。点对点消息往往与队列相关联。（2）发布/订阅（Publish/Subscribe）。发布/订阅消息系统支持一个事件驱动模型，消息生产者和消费者都参与消息的传递。生产者发布事件，而使用者订阅感兴趣的事件，并使用事件。该类型消息一般与特定的主题关联。

### AMQP和JMS的对比
<table>
	<tr>
		<th></th>
		<th>ANQP</th>
		<th>JMS</th>
	</tr>
	<tr>
		<td>定义</td>
		<td>线级协议</td>
		<td>Java API</td>
	</tr>		
	<tr>
		<td>跨平台</td>
		<td>是</td>
		<td>否</td>
	</tr>
	<tr>
		<td>跨语言</td>
		<td>是</td>
		<td>否</td>
	</tr>
	<tr>
		<td>Model</td>
		<td>
			<div align="left">
				五种消息模型 
				<ul>
					<li>Direct Exchange</li>
					<li>Fanout Exchange</li>
					<li>Topic Exchange</li>
					<li>Headers Exchange</li>
					<li>System Exchange</li>
				</ul>
				本质上，后四种消息模型和Publish/Subscribe没有差别，只是在路由机制上做了更细致的划分
			</div>
		</td>
		<td>
			<div align="left">
				两种消息模型 
				<ul>
					<li>Point-to-Point</li>
					<li>Publish/Subscribe</li>
				</ul>
			</div>
		</td>
	</tr>
	<tr>
		<td>消息类型</td>
		<td>byte[]</td>
		<td>
			<div align="left">
				五种消息类型
				<ul>
					<li>Text message</li>
					<li>Object message</li>
					<li>Bytes message</li>
					<li>Stream message</li>
					<li>Map message</li>
				</ul>
			</div>
		</td>
	</tr>
	<tr>
		<td>消息流</td>
		<td>
			Producer将消息发送到Exchange，Exchange将消息路由到Queue，Consumer从Queue中消费消息。
		</td>
		<td>
			Producer将消息发送到Queue或者Topic，Consumer从Queue或Topic中消费消息。
		</td>
	</tr>
	<tr>
		<td>综合对比</td>
		<td>AMQP定义了线级协议标准；具有跨平台、跨语言特性。</td>
		<td>MS 定义了JAVA API层面的标准；在java体系中，多个client均可以通过JMS进行交互，但是其对跨平台、跨语言的支持较差</td>
	</tr>																													
</table>

# RabbitMQ和RocketMQ的比较
<table>
	<tr>
		<th>特性</th>
		<th>RabbitMQ</th>
		<th>RocketMQ</th>
	</tr>
	<tr>
		<td>Prodocer-Comsumer</td>
		<td>支持</td>
		<td>支持</td>
	</tr>
	<tr>
		<td>Publish- Subscribe</td>
		<td>支持</td>
		<td>支持</td>
	</tr>
	<tr>
		<td>Request-Reply（请求回复）</td>
		<td>支持</td>
		<td></td>
	</tr>
	<tr>
		<td>API完备性</td>
		<td>高</td>
		<td>高</td>
	</tr>
	<tr>
		<td>多语言支持</td>
		<td>语言无关</td>
		<td>只支持Java</td>
	</tr>
	<tr>
		<td>单机吞吐量</td>
		<td>万级</td>
		<td>万级</td>
	</tr>
	<tr>
		<td>消息延迟</td>
		<td>微秒级</td>
		<td>毫秒级</td>
	</tr>
	<tr>
		<td>可用性</td>
		<td>高（主从）</td>
		<td>非常高（分布式）</td>
	</tr>
	<tr>
		<td>消息丢失</td>
		<td>低</td>
		<td>低</td>
	</tr>
	<tr>
		<td>消息重复</td>
		<td>可控制</td>
		<td></td>
	</tr>
	<tr>
		<td>文档完备性</td>
		<td>高</td>
		<td>中</td>
	</tr>
	<tr>
		<td>部署难度</td>
		<td>低</td>
		<td></td>
	</tr>
	<tr>
		<td>部署方式</td>
		<td>独立</td>
		<td>独立</td>
	</tr>
	<tr>
		<td>社区活跃</td>
		<td>高</td>
		<td>中</td>
	</tr>
	<tr>
		<td>商业支持</td>
		<td>无</td>
		<td>阿里云</td>
	</tr>
	<tr>
		<td>特点</td>
		<td>并发能力强</td>
		<td>分布式扩展设计，多种消费模式，支持上万个队列，性能好</td>
	</tr>
	<tr>
		<td>支持协议</td>
		<td>AMOP</td>
		<td>基于JMS的自定义协议（社区提供的JMS不成熟）</td>
	</tr>
	<tr>
		<td>持久化</td>
		<td>内存、文件</td>
		<td>磁盘文件</td>
	</tr>
	<tr>
		<td>事务</td>
		<td>支持</td>
		<td>支持</td>
	</tr>
	<tr>
		<td>负载均衡</td>
		<td>支持</td>
		<td>支持</td>
	</tr>
	<tr>
		<td>管理界面</td>
		<td>好</td>
		<td>有web console实现</td>
	</tr>
	<tr>
		<td>评价</td>
		<td>
			<div align="left">
				<ul>
					<li>优点
						<ul>
							<li>MQ性能好</li>
							<li>管理界面丰富</li>
							<li>支持AMQP协议，跨平台能力强</li>
						</ul>
					</li>
					<li>缺点
						<ul>
							<li>erlang语言难度较大</li>
							<li>集群不支持动态扩展</li>
						</ul>
					</li>
				</ul>
			</div>
		</td>
		<td>
			<div align="left">
				<ul>
					<li>优点
						<ul>
							<li>模型简单，接口易用</li>
							<li>在阿里有大规模的应用</li>
							<li>性能好</li>
							<li>支持多种消费</li>
							<li>开发度活跃，版本更新较快</li>
						</ul>
					</li>
					<li>缺点
						<ul>
							<li>产品较新，文档缺乏</li>
							<li>基于自己的一套协议，兼容性差</li>
						</ul>
					</li>
				</ul>
			</div>
		</td>
</table>
