---
title: RabbitMQ在Python中的安装配置和使用
date: 2023-08-10 16:18:43
tags: ["Python", "RabbitMQ"]
categories: ["技术"]
---

RabbitMQ是一个免费开源的消息队列应用，应用非常广泛，本文主要介绍RabbitMQ的安装使用和Python+pika的简单调用.

<!-- more -->

### RabbitMQ安装

 - Mac

```
brew install rabbitmq
```
 - Ubuntu

```
apt install rabbitmq-server
```

更多平台以及安装方式可以去官网查看[https://www.rabbitmq.com/download.html](https://www.rabbitmq.com/download.html)

 - 常用命令

```shell
# 查看节点状态
rabbitmqctl status
# 创建虚拟主机(也就是访问节点，默认自带 '/' 根结点，强烈建议根据业务创建自己的节点)
rabbitmqctl add_vhost 主机名
# 添加用户
rabbitmqctl add_user 用户名 密码
# 修改密码
rabbitmqctl change_password 用户名 密码
# 设置管理员
rabbitmqctl set_user_tags 用户名 administrator
# 设置虚拟主机访问权限
# rabbitmqctl set_permissions -p 虚拟主机 用户名 配置 写 读
# 启用权限 ".*"  禁用权限 "^$"
rabbitmqctl set_permissions -p 虚拟主机 用户名 ".*" ".*" ".*"
# 查看用户权限
rabbitmqctl list_user_permissions 用户名
```

 - 其他说明

可以使用`rabbitmq-plugins enable rabbitmq_management`命令开启管理插件，然后访问`http://localhost:15672`进入管理界面，**管理后台的端口是15672，消息服务的端口是5672，注意不要搞混了，很多第一次使用的人会在这里踩坑**

### Python使用RabbitMQ

 - 安装pika

```
pip install pika
```

 - 消息生产端(客户端)

```python
import pika

credentials = pika.PlainCredentials("用户名", "密码")
# heartbeat=0 设置心跳检测频率，单位秒，为0则禁用心跳，服务端与客户端维持连接不会断开
connection = pika.BlockingConnection(pika.ConnectionParameters("服务IP", "端口", credentials=credentials, heartbeat=0, virtual_host="虚拟主机"))
channel = connection.channel()
# durable=True设置队列持久化
channel.queue_declare(queue='default', durable=True)
# properties=pika.BasicProperties(delivery_mode=2)开启消息持久化
channel.basic_publish(exchange='', routing_key='default', body=json.dumps({"url": url, "data": data}, ensure_ascii=False), properties=pika.BasicProperties(delivery_mode=2))
```

 - 消费端(服务端)

```python
import pika

credentials = pika.PlainCredentials("用户名", "密码")
connection = pika.BlockingConnection(pika.ConnectionParameters("服务IP", "端口", credentials=credentials, heartbeat=0, virtual_host="虚拟主机"))
channel = connection.channel()
channel.queue_declare(queue='default', durable=True)
# auto_ack=True 自动应答消息
channel.basic_consume(queueName,on_message_callback=module.run, auto_ack=True)
channel.start_consuming()
```
