---
title: celery 笔记(1)
date: 2018-08-10 00:08:45
tags:
 -- python
 -- celery
---

项目中用到了`celery`，最近踩了一些坑。花点时间整理下有关`celery`的知识。
本文是`celery`笔记的第一篇，主要了解下`celery`中的基本概念和基本使用。

<!--more-->

### 基本概念

{% blockquote %}
Celery is an asynchrounous task queue based on distributed message passing.
{% endblockquote %}

简单来说，`celery` 是一个任务队列，支持任务的异步执行，支持一定的任务调度功能，并且也可以部署多个分布式工作节点来提高任务的执行效率。

`celery` 有很多的使用场景，但共同点可以简单概括为，就是把一些比较消耗资源的任务分发到其他的任务节点去执行，以提高整个系统的工作效率。

例如这样的场景：某web服务，用户的一个请求触发了一个邮件发送。这个时候我们可以吧这个耗时的邮件任务添加到 `celery` 的任务队列中去执行；而用户的响应就可以很快滴返回，而不用去等待邮件发送的结束。而且，我们也可以很方便地查看 `celery` 中的任务的执行进度和结果。

</br>
放一张 `celery` 架构图:
{% asset_img celery_structure.png local file %}

下边介绍几个概念：

客户端(client): 添加 `celery` 任务的程序，可能为 web 服务, 或者添加任务的脚本等。

工作节点(worker): 任务的实际执行者，一般是`celery`的服务。

Broker: 消息传输的中间设施；在 `celery` 的设计结构中，客户端添加任务到队列中，消息中转者(`broker`)把任务数据或者消息传递给工作节点; `celery` 可选的 Broker 有很多，如: `RabbitMQ`, `Redis`, 等。`RabbitMQ`提供更多高级特性，相比于`Redis`也更稳定（redis掉电丢数据）。

任务结果存储(Task Result Storage)，又叫 backend。主要用于存储任务的状态、结果和异常栈等信息。可选项很多：SQLAlchemy/Django ORM, Memcached, Redis, RPC (RabbitMQ/AMQP) 等。但对于任务执行这个并不是必须的。

### 基本用法

* 安装

[`github`项目的首页](https://github.com/celery/celery/)列出了挺多的信息，如运行环境，各种依赖的安装等等。

```python
pip install -U celery
# or
# pip install celery[librabbitmq,redis,auth,msgpack]  # 安装依赖
```

* 定义任务

```python
# tasks.py
from celery import Celery

app = Celery(
    __name__,
    broker="redis://localhost:6379/0", 
    backend="redis://localhost:6379/0"
)

@app.task
def add(x, y):
    return x + y
```

在`tasks.py`中的定义的`add`任务；


* 启动工作节点`worker`

```shell
# worker shell
celery worker -A tasks -l info


(myapp) ~/Projects/demo celery -A tasks worker -l info


celery@HackEngine.local v4.2.1 (windowlicker)

Darwin-17.7.0-x86_64-i386-64bit 2018-08-09 23:20:41

[config]
.> app:         tasks:0x10d60c400
.> transport:   redis://localhost:6379/0
.> results:     redis://localhost:6379/1
.> concurrency: 8 (prefork)
.> task events: OFF (enable -E to monitor tasks in this worker)

[queues]
.> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.add
```

将工作目录切换到任务模块，然后使用上述命令来启动一个`worker`；
然后终端会打印出配置信息，在`[tasks]`这里可看到我们已注册的任务。

* 执行任务

在任务模块新打开一个python的交互环境， 以`#python shell`标识。

```python
# python shell
>>> from tasks import add
>>> add.delay(3, 4)
<AsyncResult: 9c9abc10-db1c-4071-a762-3e23fd54e5a5>
>>>
>>> _.result
7

>>> r = add.apply_async(args=(3, 4))  # 另一种任务调用方法
>>> add(3, 4)  # 直接调用会像普通函数一样执行
7
>>> add.apply_async(args=(3, 4), countdown=10)  # 添加任务，并延时10s执行
```
上述过程是在python环境中添加了一个`add`任务，然后我们得到了一个`AsyncResult`对象；
任务的结果我们可以通过这个对象得到;

我们的工作节点`worker`会显示接收到一个任务，并且给出了任务的结果。
```
# worker shell
[2018-08-09 23:26:09,309: INFO/MainProcess] Received task: tasks.add[9c9abc10-db1c-4071-a762-3e23fd54e5a5]
[2018-08-09 23:26:09,325: INFO/ForkPoolWorker-8] Task tasks.add[9c9abc10-db1c-4071-a762-3e23fd54e5a5] succeeded in 0.011711072000025524s: 7
```

整个过程就是: 通过`#python shell`客户端添加了一个任务到broker中。
然后我们的worker从broker中获取到任务数据并异步地执行；
执行完毕后worker把结果存放在backend中；
随后客户端又通过`AsyncResult`对象从backend中获取到任务的结果。

就像上边这样，我们可以把发送邮件、大文件操作、或者音视频转码等类似的操作都放在celery任务中来做。

### 写在结尾

前边我们简单了解了`celery`的架构，知道了`client`, `broker`, `worker`, `backend`具体的作用；
也通过定义了一个简单的`add`任务来了解了`celery`的使用和任务的执行过程;

后边还有一些高级的用法，我们留到下一篇来讲。

