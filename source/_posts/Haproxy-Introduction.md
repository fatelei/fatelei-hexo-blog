title: "Haproxy Introduction"
date: 2015-08-03 21:56:29
tags: [HAProxy]
---

> HAProxy 提供高可用性、负载均衡以及基于 TCP 和 HTTP 应用的代理，支持虚拟主机，并且免费。HAProxy 特别适用于那些负载特大的web站点，这些站点通常又需要会话保持或七层处理。

<!-- more --> 

## Install HAProxy

### ubuntu 14.04
```
$ echo deb http://archive.ubuntu.com/ubuntu trusty-backports main universe | \
      sudo tee /etc/apt/sources.list.d/backports.list
$ sudo apt-get update
$ sudo apt-get install haproxy
```

## HAProxy 配置简介

HAProxy 的配置主要分为三部分：

+ 来自命令行的参数，这些参数通常会被优先使用
+ 来自`global`section的参数，这些参数通常是影响整个 HAProxy 进程的
+ 最后是`proxies`section的参数，这些区域可以使`defaults`、`listen`、`frontend`、`backend`

`global`section 处的参数，主要用于调优、以及 OS 相关的一些参数，还有访问权限相关的参数。下面主要还是围绕代理相关的一些配置进行介绍。

### Proxy 配置

在 HAProxy 中 Proxy 的配置可能是和下面的一个集合的配置相关：

+ defaults [name]
+ listen [name]
+ frontend [name]
+ backend [name]

以下的摘自 [HAProxy Manual](https://cbonte.github.io/haproxy-dconv/configuration-1.5.html#1)

> A "defaults" section sets default parameters for all other sections following
its declaration. Those default parameters are reset by the next "defaults"
section. See below for the list of parameters which can be set in a "defaults"
section. The name is optional but its use is encouraged for better readability.

> A "frontend" section describes a set of listening sockets accepting client
connections.

> A "backend" section describes a set of servers to which the proxy will connect
to forward incoming connections.

> A "listen" section defines a complete proxy with its frontend and backend
parts combined in one section. It is generally useful for TCP-only traffic.

## HTTP Example

使用 HAProxy 作为 HTTP Proxy 的例子，HAPrxoy 配置文件如下：

```
global
  log /dev/log local0 notice

defaults
  mode http
  option httplog
  option dontlognull
  timeout connect 1s
  timeout client 1s
  timeout server 1s

listen http-in
  bind *:80
  server server1 127.0.0.1:8000 maxconn 22
```

HTTP Server：

```
var http = require('http');

var server = http.createServer(function (req, res) {
  res.send('Hello World');
  res.end();
});

server.listen(8000);
```

可以使用 `haproxy -f http.cfg`启动HAProxy，然后再启动相应的 HTTP Server，执行命令
`curl http://127.0.0.1`，在控制台中会输出`Hello World`字样。


## TCP Example

使用 HAProxy 作为 TCP Proxy 的例子，HAPrxoy 配置文件如下：

```
global
  log /dev/log local0 notice

defaults
  mode tcp
  option dontlognull
  timeout connect 1s
  timeout client 1s
  timeout server 1s

listen http-in
  mode tcp
  bind *:8000
  server server1 127.0.0.1:9000 maxconn 22
```

TCP Server：

```
var net = require('net');

var server = net.createServer(function (socket) {
  socket.write('Hello World');
});

server.listen(9000);

```

可以使用 `haproxy -f tcp.cfg`启动HAProxy，然后再启动相应的 TCP Server，执行命令
`telnet 127.0.0.1 8000`，在控制台中会输出`Hello World`字样。

以上就是 HAProxy 的一个简单介绍。后面如果有更深入的研究，会在后面更新上来，：）。
