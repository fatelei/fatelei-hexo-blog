title: "Tornado 源码解读: IOLoop"
date: 2015-08-08 20:40:32
tags: [tornado, framework, python]
---

> Tornado 是 Python 的 Web Framework，并且也是一个 Async Network Library（异步网络库）。最早是用 friendfeed.com（现在已经关闭）。通过使用非阻塞网络 I/O，Tornado 可以支持数万连接，很适合用于 Long Polling（长轮询）、WebSockets 这类应用。

<!-- more -->

下面是用 Tornado 实现的一个简单的 Hello World Server：

```
#!/usr/bin/env python
# -*-coding: utf8-*-

import tornado.ioloop

from tornado.web import Application
from tornado.web import RequestHandler


class HelloWorldHandler(RequestHandler):

    def get(self):
        return self.write("Hello World")


handlers = [
    (r'/', HelloWorldHandler)
]

if __name__ == "__main__":
    app = Application(handlers)
    app.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
```

上面的代码看起来比较简单，那么 Tornado 是怎么实现支持*可以支持数万连接*的呢？用了哪些魔法呢？
下面就来一探究竟（源码解读基于 Tornado >= 4.0 以上的版本）。

上面的代码片段，在 "\__main__" Block中有这样一行代码：
```
tornado.ioloop.IOLoop.instance().start()
```
这段代码的意义非常重要，Tornado 并发模型就在这下面。

---

## tornado.ioloop.IOLoop

*IOLoop* 相关的代码在 *tornado.ioloop* 下，截取部分源码如下：
```
class IOLoop(Configurable):
    """A level-triggered I/O loop.

    We use ``epoll`` (Linux) or ``kqueue`` (BSD and Mac OS X) if they
    are available, or else we fall back on select(). If you are
    implementing a system that needs to handle thousands of
    simultaneous connections, you should use a system that supports
    either ``epoll`` or ``kqueue``.
```

可以看到 IOLoop 继承于 Configurable （该 Class 是一个工厂类），并且支持 **level triggered I/O Loop**（基于事件的 I/O 触发）。

接下来查看其 *instance* 方法的实现，源码如下：
```
    @staticmethod
    def instance():
        """Returns a global `IOLoop` instance.

        Most applications have a single, global `IOLoop` running on the
        main thread.  Use this method to get this instance from
        another thread.  To get the current thread's `IOLoop`, use `current()`.
        """
        if not hasattr(IOLoop, "_instance"):
            with IOLoop._instance_lock:
                if not hasattr(IOLoop, "_instance"):
                    # New instance after double check
                    IOLoop._instance = IOLoop()
        return IOLoop._instance
```
首先 *instance* 是一个静态方法，可以直接通过类进行调用，并且这段代码还是线程安全的。这段代码的意义是：创建了一个 *IOLoop* 的实例对象，并且赋值给 *IOLoop* 的类属性 *_instance* 。上面这段代码看上去似乎没有什么特别神奇的地方，但是不要忘了：*IOLoop* 继承自 *Configurable* 。那会不会魔法在 *Configurable* 呢？解读继续。
- - -

## tornado.util.Configurable

```
class Configurable(object):
	"""
	....
	"""

	__impl_class = None
    __impl_kwargs = None

    def __new__(cls, **kwargs):
        base = cls.configurable_base()
        args = {}
        if cls is base:
            impl = cls.configured_class()
            if base.__impl_kwargs:
                args.update(base.__impl_kwargs)
        else:
            impl = cls
        args.update(kwargs)
        instance = super(Configurable, cls).__new__(impl)
        # initialize vs __init__ chosen for compatiblity with AsyncHTTPClient
        # singleton magic.  If we get rid of that we can switch to __init__
        # here too.
        instance.initialize(**args)
        return instance
```
已经快要接近魔法了，可以看到在 *Configurable* 中重写了方法 *\__new__*。在 Python 中，创建一个实例对象，是通过方法 *\__new__*。所以执行 *IOLoop()* 时，其默认的行为已经被修改了。现在来逐行解读吧（这里的 cls 指向 IOLoop）：
```
base = cls.configurable_base()
```
这里会调用 *IOLoop* 中的方法 *configurable_base*，相关代码如下：
```
@classmethod
def configurable_base(cls):
    return IOLoop
```
所以执行完之后 *base* 的值为 *IOLoop*，接着：
```
if cls is base:
    impl = cls.configured_class()
    if base.__impl_kwargs:
        args.update(base.__impl_kwargs)
else:
    impl = cls
```
上面的 *if* 条件检查会成功，然后 *impl* 的值为，从下面的代码中获取:

### tornado.ioloop.IOLoop

```
@classmethod
def configurable_default(cls):
    if hasattr(select, "epoll"):
        from tornado.platform.epoll import EPollIOLoop
        return EPollIOLoop
    if hasattr(select, "kqueue"):
        # Python 2.6+ on BSD or Mac
        from tornado.platform.kqueue import KQueueIOLoop
        return KQueueIOLoop
    from tornado.platform.select import SelectIOLoop
    return SelectIOLoop
```

所以 *impl* 会为：*EPollIOLoop*、*KQueueIOLoop*、*SelectIOLoop*中的一个，根据不同的平台或者 Python 的版本，选择合适的 IOLoop。由于本人所用的操作系统是 Linux，并且 Python 版本是 2.7.6，所以这里的 *impl* 为 *EPollIOLoop*。

接着在 *Configurable.\__new__* 剩下的代码为：
```
instance = super(Configurable, cls).__new__(impl)
# initialize vs __init__ chosen for compatiblity with AsyncHTTPClient
# singleton magic.  If we get rid of that we can switch to __init__
# here too.
instance.initialize(**args)
return instance
```
会创建一个 *EPollIOLoop* 的实例对象，*EPollIOLoop* 是继承于 *tornado.ioloop.PollIOLoop*，然后执行  *tornado.ioloop.PollIOLoop* 中的 *initialize* 方法，然后实例对象。然后：
```
tornado.ioloop.IOLoop.instance().start()
```
这段代码就会执行定义在 *tornado.ioloop.PollIOLoop* 的 *start* 方法，分析继续。

---

## tornado.ioloop.PollIOLoop

由于 *start* 的方法中代码过长，这里就只提取最主要的代码：
```
try:
    event_pairs = self._impl.poll(poll_timeout)
    except Exception as e:
    # Depending on python version and IOLoop implementation,
    # different exception types may be thrown and there are
    # two ways EINTR might be signaled:
    # * e.errno == errno.EINTR
    # * e.args is like (errno.EINTR, 'Interrupted system call')
    if errno_from_exception(e) == errno.EINTR:
        continue
    else:
        raise

    if self._blocking_signal_threshold is not None:
        signal.setitimer(signal.ITIMER_REAL,
                         self._blocking_signal_threshold, 0)

    # Pop one fd at a time from the set of pending fds and run
    # its handler. Since that handler may perform actions on
    # other file descriptors, there may be reentrant calls to
    # this IOLoop that update self._events
    self._events.update(event_pairs)
    while self._events:
        fd, events = self._events.popitem()
        try:
            fd_obj, handler_func = self._handlers[fd]
            handler_func(fd_obj, events)
        except (OSError, IOError) as e:
            if errno_from_exception(e) == errno.EPIPE:
                # Happens when the client closes the connection
                pass
            else:
                self.handle_callback_exception(self._handlers.get(fd))
        except Exception:
                self.handle_callback_exception(self._handlers.get(fd))
    fd_obj = handler_func = None
finally:
    # reset the stopped flag so another start/stop pair can be issued
    self._stopped = False
    if self._blocking_signal_threshold is not None:
        signal.setitimer(signal.ITIMER_REAL, 0, 0)
    IOLoop._current.instance = old_current
    if old_wakeup_fd is not None:
        signal.set_wakeup_fd(old_wakeup_fd)
```

最核心的代码就是这个了 **event_pairs = self._impl.poll(poll_timeout)** ，这里实际上就是使用了[epoll](http://linux.die.net/man/4/epoll)，用 epoll 来通知上面的的应用程序，哪些 fd 可读/可写/出错，由于 epoll 本身可以很好的 scales 大量需要检测的 fd(s)，这样 Tornado Server 可以一次接受比较多的连接的请求。可以说：Tornado 本身是一个 event loop + callback 的 Web Framework。

---

后面会带来 *tornado.iostream* 模块的分析，这个也是 *Tornado* 的核心模块。
