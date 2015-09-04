title: "Css Background-position For Animation"
date: 2015-09-04 11:21:29
tags: [CSS, Animation]
---

上一篇 [文章](http://fatelei.github.io/2015/09/01/Css-background-position/) 中讲到了 `background-position` 在 `Css sprite` 中的使用，这篇文章主要讲怎么用 `background-position` 来做动画。其实动画都是一帧一帧的做出来的，每一帧就是就是一个画面。通常图片在人眼中存在的时间为 1/24 秒，如果画面播放的速度超过了人眼可以分辨的速度，那么人眼所看到的画面就动起来了。

<!-- more -->

 有这样一张背景图：

![monster](/images/monster.png)

如果把图片每帧连续的播放，就能产生动画效果，代码如下：

```
<!doctype html>
<html>
  <head>
    <title>Css Sprite</title>
    <style>
      .monsters {
        background-image: url('monster.png');
        background-repeat: no-repeat;
        height: 200px;
      }

      #monster {
        width: 190px;
        animation: monster .8s steps(10) infinite;
      }

      @keyframes monster {
        from {
          background-position: 0px;
        }

        to {
          background-position: -1900px;
        }
      }
    </style>
  </head>
  <body>
    <div id="monster" class="monsters">
    </div>
  </body>
</html>
```

实际效果可以到这里查看 [动画](https://jsfiddle.net/fatelei/j47609b1/embedded/result/)。

最后附赠一个：[刘看山小站的动画实现](https://jsfiddle.net/fatelei/hgqmh6q1/embedded/result/)，原版的在 [这里](https://liukanshan.zhihu.com)
