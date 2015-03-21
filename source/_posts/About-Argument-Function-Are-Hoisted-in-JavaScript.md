title: "About Argument & Function Are Hoisted in JavaScript"
date: 2015-03-21 16:36:36
tags: [hoisted, JavaScript]
---

在很多语言如果是使用未定义的变量，通常会造成编译错误或者运行时抛出异常。
而对于JavaScript，则不会出现问题。下面的代码你拿去运行完全没有问题：

```
function test() {
  alert(name);

  var name = 'foo';

  alert(name);
}

test();
```

[jsfiddle](http://jsfiddle.net/fatelei/1a7fugdf/)

或者：

```
test();

function test() {
    alert('foo');
}
```

[jsfiddle](http://jsfiddle.net/fatelei/srsr9t22/)


造成这些原因是都是JavaScript解析器的错，解析器会对当前作用域内的变量和函数的声明提前，
对于变量它的赋值操作则保留在原来的位置，而对于函数后面会单独说。

## `变量`被提前

比如前面提到的第一个例子，实际在test函数内部是解释期间是这样的：

```
function test() {
  var name;

  alert(name);

  name = 'foo';

  alert(name);
}
```

再来看一个例子，各位想想下面的输出会是什么？

```
var name = 'foo';

(function () {
  alert(name);

  name = 'bar';

  alert(name);
})();
```

在这里可以验证，[jsfiddle](http://jsfiddle.net/fatelei/m9utsrdm/)

以及下面的代码会输出什么？

```
var name = 'foo';

(function () {
  alert(name);

  var name = 'bar';

  alert(name);
})();
```

同样也可以在这里验证，[jsfiddle](http://jsfiddle.net/fatelei/gp4jxL45/)

提示：只对当前作用域中`声明`的变量提前。

## `函数`声明被提前

在一开始的例子中，test函数会被正确执行，所以是函数的整个定义都被提前了。
再来看下面的一个例子：

```
foo();

bar();

function foo() {
  alert('foo');
}

var bar = function() {
  alert('bar');
};
```

输出将会是：

```
alert foo
undefined is not a function
```

验证地址，[jsfiddle](http://jsfiddle.net/fatelei/gbptw0uq/)
看错误的时候，记得developer tools.


会出现这两个不同结果是因为，`函数`被提前分两种：

+ 函数声明被提前
+ 函数赋值给变量被提前

所以，实际情况是：

```
function foo() {
  alert('foo');
}

var bar;

foo();

bar();

bar = function () {
  alert('bar');
};
```

最后，为了避免被语言所坑，大家最好都是先声明在调用吧，能减少不必要的Debug时间。
