---
title: 三种go并发模式
date: 2020-05-22 00:06:45
tags: golang
---

`Go`语言最为人称道的就是好用的并发编程。`Go`的协程确实非常简单，但想写出好的并发代码还需要花点功夫。

[Go Blog](https://blog.golang.org/)有三篇介绍并发模式的系列文章, 分别介绍了`Timeout`, `Pipeline`和`Context`三种实用并发模式。作者[Sameer Ajmani](https://twitter.com/Sajma?s=20), 巨牛。虽然是多年前的文章，不过其中`Go`语言思想还是一脉相承的，值得学习。

原文链接:

1. [Go Concurrency Patterns: Timing out, moving on](https://blog.golang.org/concurrency-timeouts)
2. [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
3. [Go Concurrency Patterns: Context](https://blog.golang.org/context)


## Timeout

`channel`常用于并发通信，原生并不支持超时特性，但也很容易实现:
{% gist fb12c16262a83c5df87e7f0556d0097f %}

实际上，标准库提供了方便的`time.After`来生成延时信号的通道;
{% gist 425b4385babbf73ba5762f70e656d5f1 %}



## Pipelines & Cancellation

## Context

## Wraps
