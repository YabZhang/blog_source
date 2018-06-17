---
title: pidfile的用途
date: 2018-06-17 14:54:29
tags:
---

在`linux`系统中经常可以看到很多`pid`文件，如: `/var/run`目录下。
了解了下这类文件用途，特此记录。

<!--more-->

### pidfile

`linux`下的很多程序会在运行时创建`.pid`后缀的文件，文件的内容只有一行，程序进程号。
`pidfile`一般用于检测某程序是否已经运行，和获取某运行中程序的进程号`pid`。


检测的逻辑大致是，在程序执行时尝试获取对应pid文件互斥访问权限；
若获取成功，则程序继续执行并写入进程号；若获取失败，则退出执行；
在程序执行完毕后需要对`pidfile`进行  清理工作;

看到有些例子中，互斥访问的设计可能会埋坑，比如使用文件是否存在来判断并不准确;
Ｃ语言中的 {% link pidfile https://linux.die.net/man/3/pidfile %} 使用互斥锁的方式来保证`pidfile`的互斥访问, 会是更好的方案。

尽管如此，`pidfile`还是有不少问题。
就像`Supervisor`的文档中说的,

{% blockquote Supervisor　http://supervisord.org/introduction.html %}
Pidfiles often lie.
{% endblockquote %}


### pidfile的问题

有关`pidfile`的问题，搬运个不错的回答。{% link monitoring-a-process https://superuser.com/questions/589698/monitoring-a-process %}

*  `pid`的数量是有限的，系统复用会用进程号

所以有时候`pidfile`中进程号对应进程运行得可能是其他的程序。
由进程号获取的程序状态就不太可信了。冗余的操作是再确定程序启动时间。
但事情看起来变得麻烦了。


```
# 获取当前系统最大的pid
$ cat /proc/sys/kernel/pid_max
```

32位系统上限为`32768`, 64位系统上限为`2^22`，大约四百多万。
更详细的回答参考这篇文章 {% link  what-is-the-maximum-value-of-the-process-id https://unix.stackexchange.com/questions/16883/what-is-the-maximum-value-of-the-process-id %}


* 可能的`pidfile`清理问题

通常配置了自启动的脚本，其`pidfile`并不是脚本创建的。
而是自启动服务提供者代为创建，比如: `rc`, `systemd`等。
这样可能发生`pidfile`文件的管理和清理等问题。


### 更好的服务进程管理方式

自己的轻量级服务或者脚本推荐使用`Supervisor`。


### 其他

* 打印上一条命令的进程号

```shell
# 打印上一条命令的进程号
echo &!

# e.g
echo '123' & echo &!
which python & echo &!
```

* 杀死指定程序


``` shell
# 杀死nginx进程
kill -HUP `cat /path/to/nginx.pid`
```

{% blockquote 叔度,一楼评论 https://www.zhihu.com/question/20289583 %}
不要killall，因为一般程序可能会起多个进程，但是只有主进程会接受信号。子进程接受这些信号可能因为没有处理而出错
{% endblockquote %}
