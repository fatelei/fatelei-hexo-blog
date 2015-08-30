title: "ES6: Yield Part 1"
date: 2015-08-30 16:51:59
tags: [ES6, yield]
---
在 ES6 中引入的众多特性中，`yield` 无疑是很吸引笔者的。JavaScript 提供的 `yield` 首先让笔者联想到的是 Python 中提供的 `yield`。熟悉 Python 的同学，应该都知道，`yield` 可以用来做：包一个函数转变为 `generator`、可以使用 `yield` 实现 `cooperative task`。哪 JavaScript 中提供的 `yield` 可以用来做什么呢，后面会讲到，首先来看看 ES6 中 `yield` 的基本语法吧。

<!-- more -->

### Yield Syntax

在使用 `yield` 时，通常是配合 `generator function` 使用，首先来看下 `generator function` 是怎么定义的，如下

```
function *foo () {
  ....
}
```

和普通的 JavaScript 中的函数，唯一的区别就是在函数的名字前面多了一个 `*`。接着就是这次的重点了，语法如下：

```
yield expression // 不能是语句
```

配合起来就是这样的：

```
function *foo () {
  yield "hello world";
}
```

调用 `generator function` 和 普通函数并没有什么不同，比如对于上面的 `generator function` 直接这样使用即可：

```
var it = foo();
it.next();
it.next();
```

虽然执行了 `foo()` 但是，该函数并不会立即返回，而是这样：语句执行到 `yield` 时，首先计算 `expression`，如果其有返回值，就执行通过在执行 `it.next()` 时候返回，然后函数会在 `yield` 处挂起，等待下次调用 `it.next()` 时，函数再恢复执行，直到执行完整个函数体。

虽然上面 `generator function`  中，最后都没有写 `return`，但实际上在其函数体内也是可以使用 `return` 的：

```
function *foo() {
  yield 1;
  return 2;
}

var it = foo();
console.log(it.next()); // {value: 1, done, false}
console.log(it.next()); // {value: 2, done, true}
```
###### 注：在配合 `for ... of ...` 使用时，最后 `return` 的值会被忽略掉。


再来看一个复杂的例子，有助于理解 `yield`：

```
function *foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 2);
  return x + y + z;
}

var it = foo(5);
console.log(it.next()); // {value: 6, done: false}
console.log(it.next(12)); // {value: 6, done: false}
console.log(it.next(12)); // {value: 29, done: false}
```

上面的例子中，在执行第一个 `it.next` 时，得到了 6，这是因为 `5 + 1 = 6` 然后将表达式的值，通过 `yield` 返回。这里执行的地一个 `it.next` 并没有提供任何参数，这是因为对于 `generator function` 第一个 `next` call 时，是因为这里还没有任何 `yield` 表达式，即使传递了参数，也会被忽略掉，并没有什么意义。

`generator function` 有下面两个方法：

+  Yield Next

在上面的例子中，已经开到了 `next` 函数，该函数的作用主要是：获取 `yield ___` 执行的表达式的值，并且唤醒正在等待了 `yield`，在恢复执行时，可以通过该函数传递 `value`。

+ Yield Throw

可以通过这个函数，抛出异常 `it.throw('xx')` ，抛出的异常可以在 `generator function` 中被捕获，如果没有在 `generator function` 中被捕获，异常就会被传递到当前执行 `it.throw('xx')` 的语句处：

在 `generator function` 内部被捕获：

```
function *foo() {
   try {
      yield;
   } catch (err) {
      console.log(err); // "xx"
   }
}

var it = foo();
it.next();
it.thow("xx");
```

不在 `generator function` 中被捕获：

```
function *foo() {
  yield;
}

var it = foo();
it.next();

try {
  it.throw('xx');
} catch (err) {
  console.log(err); // 'xx'
}
```

### Yield Delegate

通过 `yield *foo` 的方式，你可以在一个 `generator function` 中执行另外一个 `generator function`。

```
function *bar() {
  yield 3;
  yield 4;
}

function *foo() {
  yield 1;/
  yield 2;
  yield *bar();
  yield 5;
}

var it = foo();
it.next();

console.log(it.next()); // {value: 1, done: false}
console.log(it.next()); // {value: 2, done: false}
console.log(it.next()); // {value: 3, done: false}
console.log(it.next()); // {value: 4, done: false}
console.log(it.next()); // {value: 5, done: true}
```

上面这个例子，就是在 `foo` 中执行了 `bar`。

以上就是一些 `yield` 的一些基本使用方法，那么问题来了，用 `yield` 究竟可以用来做什么了？先看一个例子：

```
var https = require('https');
var parse = require('url').parse;
var it;

function asyncCall (url) {
  var options = parse(url)

  var req = https.get(options, function (res) {
    var bufs = [];
    var length = 0;

    res.on('data', function (chunk) {
      bufs.push(chunk);
      length += chunk.length;
    });

    res.on('end', function () {
      var tmp = Buffer.concat(bufs, length);
      it.next(tmp.toString());
    });
  });

  req.on('error', function (e) {
    it.throw(e);
  });
};

function *request(url) {
  try {
    var rst = yield asyncCall(url);
    console.log(rst);
  } catch (err) {
    console.error(err);
  }
};

it = request('https://www.baidu.com');
it.next();
```

通过上面的例子是不是发现了，`yield` 可以使用在异步调用场景，然后不需要再使用 `callback` 了，避免 `callback hell`。关于 `yield ` 和异步调用这块，在后面文章中再接着讨论。
