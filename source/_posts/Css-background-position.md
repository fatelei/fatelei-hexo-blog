title: "Css: background-position"
date: 2015-09-01 23:53:12
tags: [css]
---

在 CSS 中提供的 `background-position` 用来设置背景图片的最初的位置。既然需要设置位置，肯定需要根据页面中的坐标确定，而这个坐标原点的确定是有 `background-origin` 来设置的。
`background-origin` 有：padding-box、border-box、content-box，三个取值，为了比较清晰的说清楚，来看下面的一个图就知道了：

<!-- more -->

 ![background-origin](/images/Selection_010.png)

`background-position` 在没有指定时，默认是 0% 和 0%，也就是从左上开始设置图片，其取值可以为：

+ 可以设置百分比：`background-position: percentage percentage`
+ 可以设置具体的数值：`background-position: length length`，length 可以为负数。
+ 可以设置 `top`、`bottom`、`left`、`right`、`center` 这五个关键字
+ 以上的取值还可以进行组合，比如：
```
background-position: left 10px right 20px;
background-position: left 10% right 20%;
background-position: 10% 20px;
```

background-position 的第一个值表示，偏离左边多少，第二个值表示偏离顶部多少。

实际演示一下吧：

```
#origin {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}

#test {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  background-position: 10px 20px;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}
```

![test01](/images/Selection_011.png)

对比发现第二个度，在原来的位置上，向左移动了10px，然后向下移动了20px。如果 `background-position` 中只指定了一个值，第二个值会默认为 `center`。来看下面的一个例子，上面的代码稍微修改下：

```
#origin {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}

#test {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  background-position: 10px;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}
```

![test02](/images/Selection_012.png)

可以看到 `background-position-y` 被设置为了 50%。

接着使用关键值 `top`、`bottom`、`left`、`right`、`center` 来设置 `background-position`，`top` 表示：距离上方 0%，`left` 表示：距离左边 0%，`right` 表示：距离左边 100%，`bottom` 表示：距离上方 100%，`center` 表示：距离左边或者上方都是 50%。实际来演示一下吧：

```
#origin {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}

#test {
  background-image: url('test.jpg');
  background-size: 200px 200px;
  background-repeat: no-repeat;
  background-position: left top;
  height: 200px;
  padding: 10px 10px;
  border: 1px solid black;
}
```

![test03](/images/Selection_013.png)

上面似乎没有讲到什么干货，都是一些比较基础的东西，接下来会是 `background-position` 在 CSS Spirte 中的应用，现在有这样一个图片：

![css sprite](/images/sprite.png)

现在我们用这一个图片，取出来不同的国旗，代码如下：

```
<!doctype html>
<html>
  <head>
    <title>Css Sprite</title>
    <style>
      .flags {
        background-image: url('sprite.png');
        background-size: 200px;
        background-repeat: no-repeat;

      }

      #first {
        background-position: -5px -5px;
        height: 112px;
      }

      #second {
        background-position: 5px -100px;
        height: 112px;
      }

      #third {
        background-position: 5px -212px;
        height: 112px;
      }
    </style>
  </head>
  <body>
    <div id="first" class="flags">
    </div>
    <div id="second" class="flags">
    </div>
    <div id="third" class="flags">
    </div>
  </body>
</html>
```

![deom](/images/Selection_015.png)

可以看到通过不管改变 `background-position` 来获取了三张不同的图片。
