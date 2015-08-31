title: "Css Draw Triangle"
date: 2015-08-31 21:05:24
tags: [CSS]
---

今天在知乎上看到这样的一个问题：[有谁能详细讲一下css如何画出一个三角形？怎么想都想不懂？](http://www.zhihu.com/question/35180018)，一下来了兴趣就去回答了。

<!-- more -->

这题里面有个答案已经回答的不错，不过感觉细节还是欠缺一些，所以笔者针对一些细节来讲解下。

首先我们来看下一个 标准的 box model 的 border 是怎样的：

![Box Model](/images/Selection_001.png)

每部分的border，只有一个小梯形，以 border-left 为例，所占的部分由两条对角线（部分），以及上下底组成。空白的中间部分由 content 和 padding 组成。接着往下看（这里我们默认 padding 都为 0px），如果我们把 width 设置为 0px 之后，这个 box model会有怎样的变化：

![no width](/images/Selection_002.png)

你会发现，怎么 border-top 和 border-bottom 已经变成一个三角形啦，然后接着把 height 设置为 0px呢？

![no height](/images/Selection_003.png)

css 画三角形的原理就是上面讲到的，所做就是需要设置 width: 0px; height: 0px，然后设置诸如合理 border-(left|top|bottom|right)-width 以及对应的颜色，然后再把你想要隐藏的隐藏就行了，题主可以试着想一下，如果没有 border-top 或者 border-bottom 或者 border-left 或者 border-right （设置相应的 width 为 0px），上面的图形会接着怎么改变，可以到这里来尝试一下 Edit fiddle - JSFiddle，还可以想下如果同时没有 border-top 和 border-bottom 会是怎么样的。

+ Css triangles

```
.left-right-triangle {
  width: 0px;
  height: 0px;
  border-left: 50px solid red;
  border-bottom: 50px solid transparent;
}
```
![left-right-triangle](/images/Selection_005.png)

```
.left-triangle {
  width: 0px;
  height: 0px;
  border-left: 50px solid red;
  border-top: 50px solid transparent;
  border-bottom: 50px solid transparent;
}
```

![left-triangle](/images/Selection_006.png)


```
.down-triangle {
  width: 0px;
  height: 0px;
  border-top: 50px solid red;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
}
```

![down-triangle](/images/Selection_007.png)

```
.up-triangle {
  width: 0px;
  height: 0px;
  border-bottom: 50px solid red;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
}
```

![up-triangle](/images/Selection_008.png)

```
.rotate-triangle {
  width: 0px;
  height: 0px;
  border-bottom: 50px solid red;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  transform: rotate(30deg);
}
```

![rotate-triangle](/images/Selection_009.png)
