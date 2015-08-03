title: "Node.js: setImmediate vs process.nextTick"
date: 2015-03-15 14:56:35
tags: [Node.js, setImmediate, process.nextTick]
---

在Node.js中，setImmediate和process.nextTick的使用，经常会有一些疑惑，
	两个函数的执行究竟有什么区别，下面就来一探究竟吧。

<!-- more --> 

	先准备两段代码：test_setimmediate.js 和 test_process_nexttick.js

test_setimmediate.js

```
	function print(number) {
	  console.log(number);
	}

	setImmediate(function A() {
	  setImmediate(function B() {
	    print(1);
	    setImmediate(function D() {
	      print(2);
	    });
	    setImmediate(function E() {
	      print(3);
	    });
	  });

	  setImmediate(function C() {
	    print(4);
	    setImmediate(function F() {
	      print(5);
	    });
	    setImmediate(function G() {
	      print(6);
	    });
	  });
	});

	setTimeout(function () {
	  console.log('Timeout out');
	}, 0);
```

test_process_nexttick.js

```
function print(number) {
  console.log(number);
}

process.nextTick(function A() {
  process.nextTick(function B() {
    print(1);
    process.nextTick(function D() {
      print(2);
    });
    process.nextTick(function E() {
      print(3);
    });
  });

  process.nextTick(function C() {
    print(4);
    process.nextTick(function F() {
      print(5);
    });
    process.nextTick(function G() {
      print(6);
    });
  });
});

setTimeout(function () {
  console.log('Timeout out');
}, 0);
```

	接下来，分别看下两段代码的执行结果，测试所用Node.js版本为v0.10.26，
	两段代码分别执行3次（不要为什么是3次，如果一定想知道，那是因为三生万物）。

	那么现在就开始吧。

首先是执行test_setimmediate.js，运行结果如下图：

![setimmediate_picture](/images/Snip20141213_39.png)


接着执行test_process_nexttick.js，运行结果如下图：

![process_nexttick](/images/Snip20141213_40.png)


	看完运行结果，对于Node.js有所了解的同学，应该能看出两个函数的区别了吧。当然如果还
	没有能够理解，那就接着往下面看。首先setImmediate和process.nextTick中的
	callback函数的执行顺序是按照进入队列的顺序执行，即FIFO。

	两者的区别是：

	1. process.nextTick

	process.nextTick中callback队列是在每次进入Node.js的Event I/O Loop之前会被
	iteration，如果队列中有callback需要被执行，被pop出要执行的callback，在执行完
	之后，再进入Node.js的Event I/O Loop中，执行还没有处理的事件。
	所以在test_process_nexttick.js的执行顺序总是：
	A -> B -> C -> D -> E -> F -> G -> setTimeout。

	2. setImmediate

	setImmediate中的callback队列则是在每次Node.js的Event I/O Loop中被
	iteration。这样做的好处是：不会让I/O产生straved。

	在setImmediate中scheduled的function的执行总是在I/O events的callbacks之后。
	所以虽然setImmediate中scheduled的callbacks的执行顺序可以保证，但是在
	这些“immediated”的callbacks中的执行中，会插入其它I/O events的回调函数被执行。

	所以在test_setimmediate.js的执行顺序中，setTimeout中delayed的function，会
	插入到setImmediate的执行顺序中。出现"Timeout out"的打印出现类似随机打印的感觉。


	嗯，就写到这儿吧。


下面是它们的两个官方说明：

+ [setImmediate](http://nodejs.org/api/timers.html#timers_setimmediate_callback_arg)
+ [process.nextTick](http://nodejs.org/api/process.html#process_process_nexttick_callback)
