---
title: celery 笔记(2)
date: 2018-08-10 00:16:33
tags:
 -- python
 -- celery
---

项目中用到了`celery`，最近踩了一些坑。花点时间整理下有关`celery`的知识。
本文是`celery`笔记的第二篇，主要是`celery`的常见高级用法。

<!--more-->


### 高级用法

在上一篇中，我们了解了`celery`的基本架构，以及作为任务队列的基本使用。
这里我们讲两个常见的高级用法，原语和任务路由。

`celery`有很多高级的用法，例如：选择`rabbitmq`作为`broker`可以利用消息队列的一些高级特性，如队列的优先级等。

但这里我们只讲一些比较常规的高级用法，并没有特别的`broker`要求的。
我在描述和做demo时均使用redis作为broker和backend。

#### 工作流原语 (workflow primitives)

* 任务签名

在说原语前，先简单说明下任务签名(signature)。

签名就像是一个把任务的参数，和执行选项与任务打包后的对象，可以直接调用和执行任务；
任务的作用是使得待执行的任务像参数一样便于传输，也便于对不同的任务工作流的操作。

```python
# 创建方法 1
>>> from celery import signature
>>> sig = signature('tasks.add', args=(3, 4))

# 创建方法 2
>>> sig = add.s(3, 4)

# 签名的调用
>>> sig.delay()  # sig.apply_async() 同样可以调用
<AsyncResult: fb893496-27f1-47a3-a04c-8bba1cdaced2>

# 不可变任务签名
>>> isig = signature('tasks.add', args=(3, 4), immutable=True)  # 不可变签名绑定的任务数据不可更改
>>> isig = add.si(3, 4)  # 不可变签名
```

任务签名更像一个包含了任务参数的高阶函数。把具体任务的声明和调用分隔开来，便于任务的调度。

* 原语与组合

原语是把多个任务的签名组合起来。

1.Group
``` python
>>> from celery import group
>>> res = group(add.s(2, 2), add.s(3, 3), add.s(4, 4))()
>>> res.get(timeout=1)  # 设置等待超时时间
[4, 6, 8]
>>> res = group(add.s(i, i) for i in range(10))()
...
```
`group`使多个任务并行执行，全部任务结束后，并把结果按照任务添加顺序返回。

2.Chain
```python
>>> res = chain(add.s(2, 2), add.s(4), add.s(8))()
>>> res.get()
16
```
`chain`把多个任务串起来，按顺序执行；并把前一个任务的结果作为参数传递到下一个任务中。
最终返回最后一个任务的结果。

3.Chord

```tasks.py
# 新增xsum任务
@app.task
def xsum(numbers):
    return sum(numbers)
```

`chord`的作用就像是`group`再加一个回调函数。
```
from celery import chord
from tasks import add, xsum

>>> chord(add.s(i, i)
          for i in xrange(100))(xsum.s()).get()
9900
```

4.Map, Starmap

```python
# 新增is_odd任务
@app.task
def is_odd(x):
	return x % 2 == 1
```

```python
>>> from tasks import is_odd
>>> is_odd.map(range(10))
[tasks.is_odd(x) for x in [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]]  # mao的作用很明显了，将每一项作为参数调用任务

>>> is_odd.map(range(10)).delay()  # 只会向broker添加一个任务！

>>> xsum.starmap([range(10), range(100)])
[tasks.xsum(*x) for x in [range(0, 10), range(0, 100)]]

>>> add.starmap([(3, 4), (5, 6)])
[tasks.add(*x) for x in [(3, 4), (5, 6)]]
>>> add.starmap([(3, 4), (5, 6)]).delay().get()
[7, 11]
```

`Map`的作用类似于`map`函数，依次对每一个参数执行任务函数；
不同的时，使用`Map`只会添加一个任务到队列中，并对单个任务反复执行的结果进行收集，效果类似于`Group`但是只在一个worker中执行；
`Starmap`的作用也类似，只是会对参数多做一次拆包处理。

5.Chunk

```python
>>> add.chunks(zip(range(0, 10), range(1, 11)), 5)
celery.chunks(task=tasks.add(), it=[(0, 1), (1, 2), (2, 3), (3, 4), (4, 5), (5, 6), (6, 7), (7, 8), (8, 9), (9, 10)], n=5)

>>> add.chunks(zip(range(0, 10), range(1, 11)), 5).delay().get()
[[1, 3, 5, 7, 9], [11, 13, 15, 17, 19]]
```

`Chunks`的作用就是把大量的任务分块添加到任务队列，每个块的数据量n由第三个参数指定；
不足的数量单独算作一块。

此外删除的原语结构可以嵌套足够，灵活地定制的任务流。

```python
@app.task
def on_chord_error(request, exc, traceback):
    print('Task {0!r} raised error: {1!r}'.format(request.id, exc))

# 通过chain把group和错误处理任务链接起来； 由on_chord_error处理可能的错误
>>> c = (group(add.s(i, i) for i in range(10)) |
...      xsum.s().on_error(on_chord_error.s())).delay()
```

#### 任务队列 (task queue)

`celery`所有未指明队列的任务都归属到默认的任务队列`celery`(在配置中可以修改默认的队列名)；
但这样把所有的任务加到同一个队列中容易出问题，如较慢的任务可能阻塞队列。
更好的做法是按照任务的类别，区分不同的任务队列，并绑定到不同的worker;

1. 路由任务到指定的队列

```python
# 方法 1： 在配置中指定
task_routes = {
        'tasks.slow_task': {
            'queue': 'slow',
            'routing_key': 'slow',
        },
}

# 方法 2: 定义任务时指定队列
@app.task(queue='slow')
def slow_task(*args):
	pass

# 方法 3：任务调用时指明队列或路由key
task_args = (1, 2, 3)
slow_task.apply_async(args=task_args, queue="slow")
```

上述三者的优先级分别为： 3 > 2 > 1，较高优先级可以覆盖较低优先级的配置;

2. 绑定队列到`worker`

不指定队列时启动worker, 其实绑定到默认的队列
```python
celery -A tasks worker -l info
# or
celery -A tasks worker -l info -Q celery  # 与上一句等价, 只执行celery队列中的任务

celery -A tasks worker -l info -Q slow  # 只执行slow队列中添加的任务

celery -A tasks worker -l info -Q slow,celery  # 可以执行两个渠道的任务
```

需要说明的时，当一个worker处理多个队列的任务时，未指定队列优先级时，获取任务的顺序是随机的。
`rabbitmq`作为broker时，支持队列配置不同的优先级。

此外, `apply_async`也支持`priority`参数，可以为同一个队列中的任务指定不同的执行优先级。

### 写在最后

在前一篇文章里，我们知道了`celery`的基本架构和基本使用。
而本篇，我们学会了使用不同的任务原语来组织任务，实现更灵活的任务流。

此外，我们知道了`celery`是按照任务队列来向worker分发任务的，而多种不同的任务放在同一个队列中是不明智的决定。
根据任务的需要可以指定不同的任务队列，开启多个worker分别取执行不同类别的任务，减少任务被阻塞的可能。
具体的部署方案，可以根据实际的需要来设计，但是一定把不同的任务分到不同的队列，在下一篇里我会分享因队列设置而遇到的问题。

在下一篇里，我将分享一些我收集到的`celery`最佳实践和一些可能遇到的实际问题。

