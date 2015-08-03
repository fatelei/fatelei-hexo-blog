title: "Gunicorn vs uwsgi"
date: 2015-07-05 21:49:15
tags: [gunicorn, uwsgi]
---

Gunicorn 和 uwsgi 都是实现了 wsgi 通信协议的server，都提供了 pre-fork 方式增加 server 并发处理能力。
现在对其做一个简单的性能测试对比，以便更好的在两者间做出选择。

<!-- more --> 

测试机器性能：

+ CPU：Intel® Core™ i5-4200U CPU @ 1.60GHz × 4
+ RAM：８GB
+ OS：ubuntu 14.04 TLS 64bit

测试Server代码app.py：

```
# -*-coding: utf8-*-

def application(env, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return [b'Hello World']
```

Gunicorn：

```
gunicorn -w 4 app:application -b 0.0.0.0:9000
```

uwsgi：

```
uwsgi --http :9000 --wsgi-file app.py --master --processes 4
```

测试用具使用 [wrk](https://github.com/wg/wrk)

测试命令：

```
wrk -t12 -cnum -d30s http://127.0.0.1:9000/
```

启动12个线程，每次测试更新并发数num，测试持续30s。
- - -

*测试结果如下：*

+ 平均响应时间

    ![latency](/images/latency.png)

+ 吞吐量

    ![req/sec](/images/req_per_sec.png)

+ 错误数

    ![error](/images/errors.png)


- - -

结论：

uwsgi在高并发下比gunicorn有更好的吞吐量和更少的错误数。
