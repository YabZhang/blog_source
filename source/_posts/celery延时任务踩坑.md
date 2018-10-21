---
title: celery延时任务踩坑
date: 2018-10-08 23:19:46
tags:
 -- celery
---

`celery`的延时任务参数`countdown`或者`eta` 清晰易懂，使用起来也很方便；但最近遇到一个坑。

<!--more-->


{% codeblock lang:python %}
# 120s 后执行任务
task.apply_async(args, countdown=120)

# 120s 后执行任务
task_dt = datetime.datetime.now() + datetime.timedelta(seconds=120)
task.apply_async(args, eta=task_dt)
{% endcodeblock %}

`celery`的延时任务调用真的很简单。


延时8小时后执行是项目的一个功能，但是实际到预定时间后任务被重复执行;
其中最主要的原因是测试不完全；但具体的原因还是技术在使用celery延时任务这里踩了坑,其实官方文档里已经有提过;

celery官方文档中有提到使用redis作为broker时，需要注意的问题；

{% blockquote %}
**Visibility timeout**

*If a task isn’t acknowledged within the Visibility Timeout the task will be redelivered to another worker and executed.*
如果一个任务没有在`visibility timeout`时间内被确认，就会被重新分发到另一个`worker`去执行.

*This causes problems with ETA/countdown/retry tasks where the time to execute exceeds the visibility timeout; in fact if that happens it will be executed again, and again in a loop.*
这会让采用了`etc/countdown/retry`这些特性并且超时没有确认的任务出问题，具体就是任务被重复地执行。

*So you have to increase the visibility timeout to match the time of the longest ETA you’re planning to use.*
所以，你必须增加`visibility timeout`的配置值来覆盖你打算使用的最长`eta`延时时间。

*Note that Celery will redeliver messages at worker shutdown, so having a long visibility timeout will only delay the redelivery of ‘lost’ tasks in the event of a power failure or forcefully terminated workers.*
需要注意的是，`celery`会在`worker`关闭的时候重新分发任务，所以配置一个较长的`visibility timeout`值只会让那些“丢失”的任务延迟执行，就比如在服务器停电或者`worker`被暴力中止时导致的任务丢失。

Periodic tasks won’t be affected by the visibility timeout, as this is a concept separate from ETA/countdown.
周期性任务不受这个配置影响，因为周期任务的原理不同于`eta/countdown`这样的延时任务。

You can increase this timeout by configuring a transport option with the same name:
你可以这样配置这个值：

{% codeblock lang:python %}
app.conf.broker_transport_options = {'visibility_timeout': 43200}  # 12h
{% endcodeblock %}

The value must be an int describing the number of seconds.
配置值必须是整数，表示总的秒数;

{% endblockquote %}

因为redis作为broker时，`visibility timeout`的默认值是一小时，所以延时任务被重复执行的问题就发生了。
每个小时未被确认的任务被重新分发到新的worker里去执行；这样到了预定的时间，就会有很多个待执行任务；
通过把`visibility timeout`减少到很短的时间，可以复现问题；而解决方法也就是把这个配置的值调到足够得大。