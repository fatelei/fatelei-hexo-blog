title: "WebSocket Introduction"
date: 2015-07-06 22:02:59
tags: [websocket]
---


WebSocket 是 HTML5 提供的在单个 TCP 连接下实现全双工的通信，IETF 标准文档为 [RFC6445](https://tools.ietf.org/html/rfc6455)。
WebsocketAPI 也被 W3C 定为标准。

在 WebSocket 之前，传统的要实现 browser - server 实时推送技术，通常使用 long-poll 技术，就是 client 不断的向 server 请求数据，每次 client 都要重新发起一个 *HTTP Request*，并且 *HTTP Header* 通常会很大，但是数据部分则是很小的，这样做的缺点也是很明显的，大大的浪费了带宽资源。

而随着 WebSocket 的出现，很方便的解决实时推送的问题，通常 client 和 server 只需要进行一次握手，就可以建立一条全双工的通信通道。

为了更直观观察 WebSocket server 和 client 的通信方式，使用 NodeJS 的 module [ws](https://github.com/websockets/ws) 中的展示的 demo server 和 client。

Server 代码如下：

```
var WebSocketServer = require('ws').Server
  , wss = new WebSocketServer({ port: 9000 });

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
  });

  ws.send('something');
});
```

Client 代码如下：

```
var WebSocket = require('ws');
var ws = new WebSocket('ws://localhost:9000/');

ws.on('open', function open() {
  ws.send('something');
});

ws.on('message', function(data, flags) {
  // flags.binary will be set if a binary data is received.
  // flags.masked will be set if the data was masked.
});
```

然后使用抓包工具 *wireshark* 进行分析。

建立连接的整个过程是：

![full](/images/full.png)

+ 首先是建立 TCP 连接

    ![tcp](/images/tcp.png)

+ 接着 Client 发起了 WebSocket 的握手协议

    ![client_handshake](/images/client_handshake.png)


+ 然后 Server 对这条 WebSocket 的握手请求，首先响应 TCP 的 ack，然后再发送 hankshake 成功的信息

    ![server_handshake_response](/images/server_handshake_res.png)
    
+  在建立 WebSocket 连接之后，开始了通信

    ![websocket](/images/websocket.png)

以上就是整个建立 WebSocket 到通信的过程，可以看出 WebSocket 还是基于 TCP 的通信协议，中间只是通过 HTTP 协议的 Server 将其 handshake interpreted 为 Upgrade request。

其中 **Sec-WebSocket-Accept** 的生成，是根据 Client 发送过来的 **Sec-WebSocket-Key** 的值，在其后拼接上 Magic String：**258EAFA5-E914-47DA-95CA-C5AB0DC85B11**，然后对拼接的字符串进行 **SHA-1** 算法hash，然后对生成的十六进制进行 Base64 编码生成，可以用 NodeJS 简单实现一个：

```
var crypto = require('crypto');

function generateSecWebSocketAccept(key) {
  var tmp = key + '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
  var sha1 = crypto.createHash('sha1');
  sha1.update(tmp);
  var d = sha1.digest('base64');
  return d;
}

console.log(generateSecWebSocketAccept('dGhlIHNhbXBsZSBub25jZQ=='));
// print s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

