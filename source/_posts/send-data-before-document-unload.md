title: 在 document unload 前几种发送数据的方式
date: 2019-07-26 18:36:39
tags: [xhr, beacon, async]
---

如果开发者希望在 `document` 完成 `unload` 前向服务器发送数据，通常不能直接发起 `asynchronous XMLHttpRequest` 在
`unload` 事件响应的回调函数中，这里因为浏览器会 `abort` 这样的请求。


通常为了实现这样的需求，有以下几种方式可供选择：

- `synchronous XMLHttpRequest`

既然浏览器会 `abort` 掉 `asynchronous XMLHttpRequest`，那我们可以考虑发起 `synchronous XMLHttpRequest` 请求，在
`unload` 或者 `beforeunload` 时间的回调函数中，这样的同步请求会阻塞住 `document unload` 的进程，直到请求的数据发送完成。

```js
window.addEventListener('unload', sendData, false);

function sendData() {
  var xhr = new XMLHttpRequest();
  xhr.open("POST", "/collect_data", false); // third parameter indicates sync xhr
  xhr.setRequestHeader("Content-Type", "text/plain;charset=UTF-8");
  xhr.send('test');
}
```

效果如下：

![sync_xhr_xmlhttprequest](/images/sync_xhr_xmlhttprequest.png)


- `fake img element`

实现上述需求，也可以通过在 `unload` 或者 `beforeunload` 时间的回调函数中创建一个 `img element`，并将数据通过指定图片 `src` 的方式将数据发送到后端。这是因为大多数浏览器会延迟 `document` 的 `unload` 进程直到完成所有未完成加载的图片完成加载。

```js
window.addEventListener('unload', sendData, false);

function sendData() {
  var img = document.createElement('img');
  img.src = 'http://example.com/collect_data.jpg?data=test'
  document.body.append(img);
}
```

效果如下：

![fake_img](/images/fake_image.png)

- `navigator.sendBeacon`

前两种方式，都存在着一个问题：由于阻塞了 `document` 的 `unload` 进程，这样就会影响到下一个页面的加载（相关数据收集 API 出现性能问题 or 网络时延）。

`navigator.sendBeacon` 的出现，正好解决了这个问题。其定义如下：

```
can be used to asynchronously transfer a small amount of data over HTTP to a web server.（可以异步地通过 `HTTP` 向服务端发送小数量的数据。）
```

`navigator.sendBeacon` 的调用，并不是直接将数据发给后端，而是将待发送的数据放入浏览器待传输队列，浏览器再将待传输的数据发送到
后端，这样就不会阻塞 `document` 的 `unload` 进程，用户也不会长时间等待。

```js
window.addEventListener('unload', sendData, false);

function sendData() {
  navigator.sendBeacon('/collect_data', 'test');
}
```

效果如下：

![fake_img](/images/sendbeacon.png)

PS: 这里虽然 `chrome` 的 `network` 面板显示请求处于 `pending` 状态，但是请求已经发送到了后端，只是还未得到响应，作者没有深入探究，猜测和在 `unload` 回调中执行有关。

##### 参考链接
- https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon

##### 相关代码

- https://github.com/fatelei/unload_send_data
