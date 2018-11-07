---
title: 服务器测试工具 —— ab
date: 2018-11-07 23:04:58
tags:
---

`ab`是Apache旗下一款知名的服务器测试工具，最适合用于测试服务器的并发性能。

`ab`的使用相当简单，下面来看下最基本的使用吧。

```bash
> ab -n 10 www.ishuhui.com/
...

Benchmarking www.ishuhui.com (be patient).....done


Server Software:        Tengine
Server Hostname:        www.ishuhui.com
Server Port:            80

Document Path:          /
Document Length:        278 bytes

Concurrency Level:      1
Time taken for tests:   0.324 seconds
Complete requests:      10
Failed requests:        0
Non-2xx responses:      10
Total transferred:      5521 bytes
HTML transferred:       2780 bytes
Requests per second:    30.85 [#/sec] (mean)
Time per request:       32.412 [ms] (mean)
Time per request:       32.412 [ms] (mean, across all concurrent requests)
Transfer rate:          16.63 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       12   15   2.9     14      20
Processing:    11   17   8.5     16      40
Waiting:       11   17   8.5     15      40
Total:         23   32  10.2     30      59

Percentage of the requests served within a certain time (ms)
  50%     30
  66%     32
  75%     34
  80%     36
  90%     59
  95%     59
  98%     59
  99%     59
 100%     59 (longest request)
```

`ab -n 10 www.ishuhui.com/` 这条命令向`ishuhui`(知名汉化组，安利一波~)连续发10个请求；
并根据响应生成结果报表；从报表中，我们可以看到`ishuhui`服务器的信息，以及我们的10个请求的响应数据统计；

`Concurrency Level` 这里指并发量，这里为1, 就是只有一个并发；

整个测试耗时`0.324`秒，所有的请求均成功；平均每个请求时间在`30.412`毫秒；

`Connection Times`部分统计了所有请求的从建立连接，到等待响应以及收到响应的时间分布；

最后的部分对所有的响应返回时间，按百分比列出来；
从中我们可以很清楚地看出，80%以上的请求在36毫秒内完成，最慢的请求耗时59毫秒。

`ab`支持非常丰富的参数。例如常见的有:

`-c` 指定并发数；未指定时默认为1;
`-n` 指定总的请求数；
`-k` 保持与服务器的长连接, 声明`Keep-Alive`;
`-H` 指定请求头部；
`-m` 指定请求方法;
`-t` 指定持续测试的时间，未指明n时默认为50000；

更多详细的说明请参见文档：https://httpd.apache.org/docs/2.4/programs/ab.html

我最常用的就是`ab -k -c 5 -n 100 www.example.com/`。
