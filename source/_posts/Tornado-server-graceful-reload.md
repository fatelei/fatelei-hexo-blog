title: Tornado server graceful reload
date: 2016-06-17 14:09:59
tags: [tornado, framework, python, graceful]
---

[tornado](https://github.com/tornadoweb/tornado) 配合 [gunicorn](https://github.com/benoitc/gunicorn) 或者 [uwsgi](https://github.com/unbit/uwsgi) 时，可以很容易的实现优雅重启（graceful reload）。但是所在公司使用了[sockjs tornado](https://github.com/mrjoes/sockjs-tornado)（模拟 WebSocket 的服务端消息推送框架），都是使用独立的进程运行在生产环境，之前想过使用 gunicorn 或者 uwsgi 部署单个工作进程跑，但是这并不是最好的解决方案。

<!-- more -->

在 google 了之后，发现 google groups 有类似的讨论帖：

 - [jveZAfNNAPo](https://groups.google.com/forum/#!topic/python-tornado/jveZAfNNAPo)
 - [VpHp3kXHP7s](https://groups.google.com/forum/#!topic/python-tornado/VpHp3kXHP7s)

大体的思路就是，监听 kill 命令发出的信号，当收到 `graceful reload` 的信号时，server 停止接受新的请求，然后 server 会等待一个固定时间（比如 10s），等待还未处理完的请求结束掉，如果超过 10s，还有未处理完的请求，server 会直接丢弃，强制重启。

举个栗子：

```
# -*- coding: utf8 -*-
"""Tornado graceful reload demo."""

import signal
import time

import tornado.web
import tornado.ioloop


class DemoHandler(tornado.web.RequestHandler):
    """Demo handler."""

    def get(self):
        self.write("hello world")


def run():
    app = tornado.web.Application([
        (r"/", DemoHandler)
    ])
    server = app.listen(8888)

    max_wait = 10  # wait for 10 seconds.

    def graceful_reload(signum, frame):
        server.stop()
        instance = tornado.ioloop.IOLoop.current()

        deadline = int(time.time()) + max_wait

        def terminate():
            now = int(time.time())
            if now < deadline and \
               (instance._callbacks or instance._timeouts):
                instance.add_timeout(now + 1, terminate)
            else:
                instance.stop()
        terminate()

    signal.signal(signal.SIGTERM, graceful_reload)
    signal.signal(signal.SIGHUP, graceful_reload)

    tornado.ioloop.IOLoop.current().start()
    return server


if __name__ == '__main__':
    run()
```

上面的代码在 `graceful reload` 时，调用了 `server.stop`，之后不再接受新的请求，其后的`terminate` 函数，则会检查是否还有未处理完的请求，或者是否到了需要强制`stop`的时间。不过上面的代码，只是做了 `stop`的事情，还没有完成启动的动作。这里有两个思路：

- 借助于 [supervisor](http://supervisord.org/index.html)，可以通过配置，设置程序的`stopsignal`为优雅重启的信号，并且配置`stopwaitsecs`（因为 supervisor 会在超过 `stopwaitsecs` 之后，会使用 `SIGKILL`强制杀掉子进程。

- 还有就是使用 `tornado` 中`autoreload.py` 中，以下的代码：

```
try:
    os.execv(sys.executable, [sys.executable] + sys.argv)
except OSError:
    # Mac OS X versions prior to 10.6 do not support execv in
    # a process that contains multiple threads.  Instead of
    # re-executing in the current process, start a new one
    # and cause the current process to exit.  This isn't
    # ideal since the new process is detached from the parent
    # terminal and thus cannot easily be killed with ctrl-C,
    # but it's better than not being able to autoreload at
    # all.
    # Unfortunately the errno returned in this case does not
    # appear to be consistent, so we can't easily check for
    # this error specifically.
    os.spawnv(os.P_NOWAIT, sys.executable,
              [sys.executable] + sys.argv)
    # At this point the IOLoop has been closed and finally
    # blocks will experience errors if we allow the stack to
    # unwind, so just exit uncleanly.
os._exit(0)
```

个人感觉还是配合 `supervisord` 使用起来简单一些。