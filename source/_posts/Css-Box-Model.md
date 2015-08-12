title: "Css Box Model"
date: 2015-08-11 23:09:18
tags: [css]
---

打开 chrome 或者 firefox 的 inspector 工具，通常你会看到这样的图形：

<!-- more -->

![css box model](/images/css_box_model.png)

上图，红色标注的区域，放大之后是这样的：

![enlarge](/images/enlarge_css_box_model.jpg)

从外到内依次是：盒模型的外边距（margin）、盒模型的边框（border）、盒模型的内边距（padding）、盒模型的内容区域（content），所以下面的计算公式：

+ 盒模型 Width = margin-left-width + border-left-width + padding-left-width + content-width + padding-right-width + border-right-width + margin-right-width
+ 盒模型 Height = margin-bottom-height + border-bottom-height + padding-bottom-height + content-height + padding-top-height + border-top-height + margin-top-height

在没有显示指定 margin 时，一般浏览器会设置一个默认的 margin width，以 chrome 为例，这个值为 8px。

## 默认 width

### Block Element

对于 block element（块级元素），默认的 width 是 100%。

测试代码
```
<!DOCTYPE html>
<html>
<head>
    <title>Css Box Model</title>
</head>
<body>
    <div id="block-box" style="background-color:red">
        This is block box
    </div>
</body>
</html>
```

在浏览器打开会看到红色的一条，占满屏幕。

### Inline Block Element

对于 inline block element（内联块级元素），默认的 width 为内容的宽度，修改上面的测试代码：

```
<!DOCTYPE html>
<html>
<head>
    <title>Css Box Model</title>
</head>
<body>
    <div id="block-box" style="background-color:red; display:inline-block">
        This is block box
    </div>
</body>
</html>
```

会看到红色的区域只包括内容宽度。
