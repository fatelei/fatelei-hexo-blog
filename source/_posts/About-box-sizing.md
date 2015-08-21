title: "About box-sizing"
date: 2015-08-21 21:59:59
tags: [Css]
---

最近参加了网易的前端面试，在面试的时候，被问了这样一些关于 CSS 盒模型相关的问题，问了什么是盒模型，以及在盒模型计算高度和宽度，由于最近才复习过相关的知识，所以很顺利的回答出来了。但是后面问了这样一道问题：有这样一块 div 其 content 的 宽度是 600px，怎么做到无论如何改变这个 div 的 margin 或者 padding 或者 border，都不改变 div 表现出来的大小？

<!-- more -->

当时没有能够回答出来，经过面试官的提示：让我后来看下 `box-sizing` 这个 CSS 属性。

[box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing) 用来改变默认的计算 *CSS Box Model* 的高度和宽度的行为。

box-sizing 有下面三个值：

+ content-box

```
width 和 height 只是 content 区域的，不把 margin、padding、border 包括在内
```

+ padding-box

```
width 和 height 包括 padding，不把 margin、border 包括在内
```

+ border-box

```
width 和 height 包括 padding、border，不把 margin 包括在内
```

有下面的代码：

```
<html>
  <head>
    <style>
      #small {
        width: 600px;
        border: 10px solid #666666;
        margin: 20px;
      }
    </style>
  </head>
  <body>
    <div id="small">
    </div>
  </body>
</html>
```

可能在没有接触盒模型的时候，你可能上面的 div 会显示出 600px 的数据吧，但是实际上却不是，因为有 *margin*、*padding*、*border* 这样会撑开元素，所以你只想要 600px 的时候，你可能要做一些使用盒模型的计算公式，计算出合适的 *width* 。现在有了 *box-sizing* ，只需要这样：

```
#small {
  width: 600px;
  border: 10px solid #666666;
  margin: 20px;
  -moz-box-sizing: border-box;
       box-sizing: border-box;
}
```

这样可见的区域就是 600px 了。
