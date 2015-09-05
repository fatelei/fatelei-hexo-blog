title: "Css: z-index"
date: 2015-09-05 20:38:19
tags: [css, z-index]
---

CSS 中提供的属性 `z-index` 用来表示某个元素和它的祖先在 `z` 数轴上的顺序。当元素重叠时，通过它们在 `z` 数轴上的顺序，决定哪个元素可以覆盖哪个元素。这个优先级是根据 `z-index` 的取值来决定的，值越大优先级越高。

<!-- more -->

说到重叠，我们可以使用 `z-index` 很容易实现遮罩和浮层的效果。

+ 遮罩

代码如下：

```
<!doctype html>
<html>
  <head>
    <title>Css Sprite</title>
    <style>
      html, body {
        margin: 0;
        padding: 0;
        height: 100%;
      }

      .overlay {
        width: 100%;
        height: 100%;
        z-index: 10;
        position: fixed;
        top: 0;
        left: 0;
        background-color: rgba(0, 0, 0, 0.25);
      }

      #content {
        margin: 300px auto;
        max-width: 300px;
      }
    </style>
  </head>
  <body>
    <div id="content">
      这里有一个遮罩。
    </div>
    <div class="overlay">
    </div>
  </body>
</html>
```

实际效果在[这里](https://jsfiddle.net/fatelei/eycefyne/)，这里通过 `z-index` 设置一个透明遮罩。

+ 浮层

代码如下：

```
<!doctype html>
<html>
  <head>
    <title>Css Sprite</title>
    <style>
      html, body {
        margin: 0;
        padding: 0;
        height: 100%;
      }

      .overlay {
        width: 100%;
        height: 100%;
        z-index: 10;
        position: fixed;
        top: 0;
        left: 0;
        background-color: rgba(0, 0, 0, 0.25);
      }

      #modal {
        max-width: 300px;
        background-color: white;
        border: 1px solid black;
        z-index: 11;
        position: fixed;
        top: 50%;
        left: 50%;
        text-align: center;
        padding: 50px 50px;
        margin-left: -100px;
        margin-top: -100px;
        box-shadow: 0 0 0 9999px rgba(0,0,0,0.5);
      }

      #content {
        margin: 300px;
      }
    </style>
  </head>
  <body>
    <div id="modal">
      这里有一个浮层。
    </div>
    <div class="overlay">
    </div>
    <div id="content">
      Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sint blanditiis perspiciatis nesciunt possimus minus molestiae culpa necessitatibus atque ut eveniet id magnam delectus reprehenderit! Ad atque aperiam rerum quas vitae!Lorem ipsum dolor sit amet, consectetur adipisicing elit. Velit esse nihil iusto ea natus aliquam enim ducimus deleniti vitae quibusdam aperiam voluptatibus necessitatibus nulla id non consectetur commodi! Laborum nam.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Dolorem non quo laudantium doloremque necessitatibus deserunt sed vero officiis iste asperiores quasi quae nisi ab debitis dignissimos delectus cum repellat quidem.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Modi a deserunt nostrum eos magni porro vel error dolore cupiditate iure sint fugiat rerum accusamus voluptates tenetur doloremque debitis ad reprehenderit?Lorem ipsum dolor sit amet, consectetur adipisicing elit. Repellendus aut dolorum officia eos commodi sit cumque nulla impedit perspiciatis ipsum odit expedita harum praesentium ducimus id labore deserunt quo repellat.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Odit in voluptas excepturi perspiciatis iste vel laborum sunt nostrum quaerat laboriosam enim amet molestiae fugiat quos illum recusandae cum asperiores. Doloribus.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Consectetur blanditiis repudiandae voluptatum provident quaerat fugiat tenetur. Voluptates soluta unde saepe necessitatibus mollitia culpa itaque tenetur alias commodi omnis fugit corrupti!Lorem ipsum dolor sit amet, consectetur adipisicing elit. Tempora expedita reiciendis facilis eius ea pariatur at dolore aperiam reprehenderit perferendis non ipsa quidem inventore unde architecto veniam similique amet commodi.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Obcaecati amet veritatis architecto a eos corrupti rerum cumque nemo distinctio ut necessitatibus doloribus tempore aliquam vero non facilis illum optio qui.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Alias necessitatibus tempore error eaque quidem id nulla veritatis quas iste eveniet aliquid ut cumque suscipit sapiente totam voluptatem cum eum adipisci.Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt dolores iste maxime qui adipisci eius placeat. Unde debitis veniam doloremque quae magni inventore temporibus dignissimos culpa earum ipsa possimus repudiandae!
    </div>
  </body>
</html>
```

实际效果在[这里](https://jsfiddle.net/fatelei/8bn20qpb/)
