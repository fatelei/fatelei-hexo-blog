title: Go tcp programming
date: 2017-09-24 19:59:05
tags: [Golang, Tcp]
---

在网络编程，无论哪种编程语言，本身对于学习这门语言的用户，都有着很强的吸引力，那今天来看看 [Go](https://golang.org/) 的网络编程是个什么样的。
这里先给出一个 ping-pong 的 tcp server & tcp client 的例子。

<!-- more -->

- server

```golang
package main

import (
	"fmt"
	"net"
)

func handle(conn net.Conn) {
	for {

	}
}

func main() {
	server, err := net.Listen("tcp", ":8000")
	if err != nil {
		panic(err)
	}

	for {
		client, err := server.Accept()
		if err != nil {
			panic(err)
		}

		var data = make([]byte, 1024)
		count, err := client.Read(data)

		if err != nil {
			panic(err)
		}

		if count > 0 {
			fmt.Printf("%s\n", string(data[:count]))
		} else {
			fmt.Println("nothing")
		}

		client.Write([]byte("pong"))
	}
}
```

- client

```golang
package main

import (
	"fmt"
	"net"
)

func main() {
	client, err := net.Dial("tcp", ":8000")
	if err != nil {
		panic(err)
	}

	client.Write([]byte("ping"))
	var data = make([]byte, 1024)
	count, err := client.Read(data)
	if err != nil {
		panic(err)
	}

	if count > 0 {
		fmt.Printf("%s\n", string(data[:count]))
	} else {
		fmt.Println("nothing")
	}
	client.Close()
}
```

以上两段实现了 client 向 server 发送 ping，server 向 client 响应 pong。

- server 端

1. 调用 `net.Listen` 创建了服务端的 `socket`, `net.Listen` 接收两个参数，一个表示协议类型，另一个表示监听的地址和端口；
2. 用一个死循环，让 server 去不断的从 accept 队列中，获取已经创建好的 `tcp` 连接；
3. 如果有来自客户端连接，就调用 `Read` 方法读取来自客户端的数据，这里 `Read` 接收的参数类型为 `[]byte`，特别注意，需要给 byte 数组设置长度，不然读不出来数据；
4. 打印读出来的数据后，再调用 `Write` 方法，发送 pong 给客户端。

- client 端

1. 调用 `net.Dial` 创建客户端套接字，参数和 `net.Listen` 一样；
2. 剩下的步骤和 server 端一致。

可以说在 golang 中实现 tcp server & client，比 c 语言简单了不少。


### 参考链接

- https://golang.org/pkg/net/