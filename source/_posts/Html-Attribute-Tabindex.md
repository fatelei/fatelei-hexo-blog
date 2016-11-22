title: Html Attribute Tabindex
date: 2016-11-22 21:49:54
tags: [tabindex]
---

`tabindex` 作为可以作用于所有 `Html` 元素上面的属性，其值的类型为整型，其作用是表明一个元素是否
能够 `focusable`（通常 `div` 是不能被 `focus` 的，但是可以通过 `tabindex`，让其可以 `focusable`）、
可以通过键盘进行导航，以及通过键盘进行导航时候的顺序。

<!-- more -->

通常 `tabindex` 的取值可以有以下三种：

- 可以小于 0

表示元素可以 `focusable`，但是只能通过 `javascript`，不能通过键盘进行导航。

- 可以为 0

表示元素可以 `focusable`，并且也能通过键盘进行导航，但是其导航的相对顺序由所在的平台决定。

- 可以大于 0

和 `tabindex = 0` 时一样，但是其导航的相对顺序由其值的大小确定，按照升序导航；当整个文档中有两个的元素的值相同时（都大于0），
其导航顺序由其在整个文档中的排列的顺序进行确定。

一个例子，`no tabindex & tabindex < 0 & tabindex = 0`：

<script async src="//jsfiddle.net/fatelei/dfh5zbyt/9/embed/js,html,css,result/dark/"></script>

另外一个例子，`tabindex > 0`：

<script async src="//jsfiddle.net/fatelei/njp7qjrr/2/embed/js,html,css,result/dark/"></script>

