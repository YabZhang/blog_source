---
title: celery 笔记(3)
date: 2018-08-11 20:27:59
tags:
 -- python
 -- celery
---

项目中用到了`celery`，最近踩了一些坑。花点时间整理下有关`celery`的知识。本文是`celery`笔记的第三篇，主要记录下`celery`的最佳实践。
<!--more-->

### 最佳实践

最佳实践主要源自一份网络博文, [传送门](https://denibertovic.com/posts/celery-best-practices/)。中文翻译版，辅助理解，[传送门](http://www.cnblogs.com/ajianbeyourself/p/3889017.html#_label4)。

* 不要使用数据库作为

作者说有不少人为了方便会使用数据库作为`broker`使用。但这样容易出问题，在性能上会有瓶颈，不利于拓展；也有可能影响到其他服务。例如：较多的工作节点可能会耗尽数据库连接；

* 使用多个任务队列

未指明队列的任务会使用默认的任务队列(可配置，不配置时默认是`celery`)。不同类型的任务最好分不同的任务队列，如：订单相关的任务和邮件发送，明显优先级不同，要分开处理。

* 使用优先级`worker`

这条作者认为，为任务较多的队列部署更多的worker节点，使得量大的任务可以得到有效的处理。参见上一篇文章部署不同队列worker的方案。

此外，`celery`支持为同意队列的任务配置不同优先级, 高优先级的任务能先被处理。`celery`还支持定义优先级队列，场景是在某worker绑定多个任务队列时，优先处理较高优先级队列的任务，但只有少量`broker`支持，如rabbitmq，redis不支持。

* 使用`celery`的错误处理机制

建议使用`celery`的错误处理机制来处理失败等情况，例如：任务重试。

* 使用`Flower`

`Flower`是一个`celery`任务监控工具，可以方便地看到`broker`和`worker`的任务进度和实际的执行情况。

* 忽略任务的结果，除非你真的需要

很多的任务并不需要返回数据。任务状态也只有做统计时才用到。除非真的需要，一般可以忽略任务结果，具体配置为`CELERY_IGNORE_RESULT = True`。

* 不要在任务调用中传参数据库对象或者orm对象

主要是针对`python`的项目实践，传参数据库对象，最后任务中拿到的有可能已经是旧的数据。更好的做法是传数据条目的id。

### 可能的问题
1. 通过`task_id`获取任务结果

每个任务都有一个唯一的`task_id`，可以由此获取任务的执行状态和执行的结果;使用`celery.result.AsyncResult`是一种较为通用的任务结果获取方式;
```python
from tasks import app
from celery.result import AsyncResult

task_id = "a9bab539-5c7a-4b24-8b19-1aea619cb6d5"  # mock task_id
task_result = AsyncResult(task_id, app=app)

print("status: ", task_result.status)
print("result: ", task_result.get(timeout=1))  # 1s timeout
```
上述的`app`是声明的celery实例，在这里是必须的；
若缺失则会因为缺失配置上下文(例如：broker等)而获取任务结果失败;

更简单的获取方式，但需要明确任务的类型。
```python
from tasks import add

task_id = "a9bab539-5c7a-4b24-8b19-1aea619cb6d5"  # mock task_id
result = add.AsyncResult(id="copied_task_id")
```

2. 合理配置任务队列和`worker`，添加任务监控

根据可能任务量级配置足够的worker来处理任务，否则可能导致任务队列的任务堆积。最好可以添加任务的监控，例如`flower`等工具;

3. db session的更新问题

这是我在`flask`项目中遇到的一个问题。任务中的db session异常，如：任务中的session无法查到新插入的数据；应该是和数据库的隔离级别有关。

```python
 @task_postrun.connect
 def close_session(*args, **kwargs):
     # this ensures tasks have a fresh session (e.g. session errors won't propagate across tasks)
     db.session.remove()
```

### 写在最后

本文罗列了一些`celery`最佳实践, 并对可能会出的问题给出了解答。基本总结下来就是采用推荐配置，保持任务简单，区分不同的任务队列，添加监控;


