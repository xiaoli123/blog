title: 你不需要 js 就可以实现一个轮播
date: 2016-08-10 18:44:47
tags:
- CSS
categories: [CSS]
bgimage: http://fakeimg.pl/2000x800/0079D8/fff/?text=Without-JS
---

# Tip
轮播这种组件是大部分网站都会存在的了，绝大部分都是 js 来实现逻辑控制的，它的原理就不多阐述了，因为我们现在要做的是不用 js 来实现一个轮播组件！

前提是只兼容现代浏览器。

# 原理

这里我们主要用的原理：

- CSS3 element+element 选择器（相邻兄弟选择器），`` element+element`` 选择器用于选取第一个指定的元素之后（不是内部）紧跟的元素。

- CSS3 element1~element2 选择器，``element1~element2`` 选择 ``element1`` 之后出现的所有 ``element2。两种元素必须拥有相同的父元素，但是`` ``element2`` 不必直接紧随 ``element1``。

- CSS3 :checked 选择器，``:checked`` 选择器匹配每个选中的输入元素（仅适用于单选按钮或复选框）。

- HTML5 `label` 标签的 `for` 属性。

我们能实现的效果：
- 轮播图片的前进后退
- 选择某张图片
- 图片切换的淡入淡出

我们实现不了的：
- 自动轮播
- 滑动的轮播切换

# 开干

1. 首先对于一个轮播组件，我们需要每个图片的容器（废话）:

```html
<div class="carousel">
  <div class="carousel-inner">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/0079D8/fff/?text=Without">
    </div>
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/DA5930/fff/?text=JavaScript">
    </div>
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/F90/fff/?text=Carousel">
    </div>
  </div>
</div>
```

2. 然后我们需要只显示一张图片，并且加点样式。

```html
<div class="carousel">
  <div class="carousel-inner">
    <input class="carousel-open" type="radio" id="carousel-1" name="carousel" aria-hidden="true" hidden="" checked="checked">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/0079D8/fff/?text=Without">
    </div>
    <input class="carousel-open" type="radio" id="carousel-2" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/DA5930/fff/?text=JavaScript">
    </div>
    <input class="carousel-open" type="radio" id="carousel-3" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/F90/fff/?text=Carousel">
    </div>
  </div>
</div>
```

```css
html, body {
  margin: 0px;
  padding: 0px;
  background: #fff;
}
.carousel {
    position: relative;
    box-shadow: 0px 1px 6px rgba(0, 0, 0, 0.64);
    margin-top: 26px;
}
.carousel-inner {
    position: relative;
    overflow: hidden;
    width: 100%;
}
.carousel-open:checked + .carousel-item {
    position: static;
    opacity: 100;
}
.carousel-item {
    position: absolute;
    opacity: 0;
    -webkit-transition: opacity 0.6s ease-out;
    transition: opacity 0.6s ease-out;
}
.carousel-item img {
    display: block;
    height: auto;
    max-width: 100%;
}
```

这里我们用到 CSS3 的 `+` 兄弟选择器和 `:checked` 选择器，在 `radio` 的 `input` 有个 `:checked` 为了，当它的 `checked=true` 时会匹配，所以我们给每一个 `carousel-item` 都配上了一个 `radio`，并且 `name` 都是一个，这样就只能一组 `radio` 中选一个了。

同时我们吧 `radio` 放在每一个  `carousel-item` 之前，这样就可以用 `+` 选择器来匹配了，首先所有的 `carousel-item` 的 `opacity` 默认为 `0`，`position` 为 `absolute`；

在它之前的那个 `radio` 被checked的时候，即 `.carousel-open:checked + .carousel-item`，把 `opacity` 变为 `100`，加上过渡的 `transition`；`position` 变为 `static`。

这样，在每次选择一个 `radio` 的时候他之后的第一个 `carousel-item` 就会淡入，而先前的会淡出。

我们还需要把 `radio` 的 `input` 的  `aria-hidden` 设为 `true`，因为我们总不希望在页面上看到几个 `radio` 吧。

然而这样的话，我们如何点击操作 `radio` 呢？

3. 前后控制

```html
<div class="carousel">
  <div class="carousel-inner">
    <input class="carousel-open" type="radio" id="carousel-1" name="carousel" aria-hidden="true" hidden="" checked="checked">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/0079D8/fff/?text=Without">
    </div>
    <input class="carousel-open" type="radio" id="carousel-2" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/DA5930/fff/?text=JavaScript">
    </div>
    <input class="carousel-open" type="radio" id="carousel-3" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/F90/fff/?text=Carousel">
    </div>
    <label for="carousel-3" class="carousel-control prev control-1">‹</label>
    <label for="carousel-2" class="carousel-control next control-1">›</label>
    <label for="carousel-1" class="carousel-control prev control-2">‹</label>
    <label for="carousel-3" class="carousel-control next control-2">›</label>
    <label for="carousel-2" class="carousel-control prev control-3">‹</label>
    <label for="carousel-1" class="carousel-control next control-3">›</label>
  </div>
</div>
```

加点样式：
```css
.carousel-control {
    background: rgba(0, 0, 0, 0.28);
    border-radius: 50%;
    color: #fff;
    cursor: pointer;
    display: none;
    font-size: 40px;
    height: 40px;
    line-height: 35px;
    position: absolute;
    top: 50%;
    -webkit-transform: translate(0, -50%);
    cursor: pointer;
    -ms-transform: translate(0, -50%);
    transform: translate(0, -50%);
    text-align: center;
    width: 40px;
    z-index: 10;
}

.carousel-control.prev {
    left: 2%;
}

.carousel-control.next {
    right: 2%;
}

.carousel-control:hover {
    background: rgba(0, 0, 0, 0.8);
    color: #aaaaaa;
}

#carousel-1:checked ~ .control-1,
#carousel-2:checked ~ .control-2,
#carousel-3:checked ~ .control-3 {
    display: block;
}
```
这里用到了 `label` 标签和它的 `for` 属性，这个属性可以指定 `label` 的指向，`for` 属性接受的是需要指向的 `input radio` 的 `id`，这样只要我们点击 `label` 就可以触发 `input radio` 的点击了。

这里我们给每个 `radio` 指定了一组前后的按钮，同时用 css 来排布，每次显示当前显示图片前后图片的 `input radio` 对应的 `label`。

3. 更多的控制

我们还想要很多轮播下面都有的索引点，这样就可以选择某个索引的图片了：

```html
<div class="carousel">
  <div class="carousel-inner">
    <input class="carousel-open" type="radio" id="carousel-1" name="carousel" aria-hidden="true" hidden="" checked="checked">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/0079D8/fff/?text=Without">
    </div>
    <input class="carousel-open" type="radio" id="carousel-2" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/DA5930/fff/?text=JavaScript">
    </div>
    <input class="carousel-open" type="radio" id="carousel-3" name="carousel" aria-hidden="true" hidden="">
    <div class="carousel-item">
      <img src="http://fakeimg.pl/2000x800/F90/fff/?text=Carousel">
    </div>
    <label for="carousel-3" class="carousel-control prev control-1">‹</label>
    <label for="carousel-2" class="carousel-control next control-1">›</label>
    <label for="carousel-1" class="carousel-control prev control-2">‹</label>
    <label for="carousel-3" class="carousel-control next control-2">›</label>
    <label for="carousel-2" class="carousel-control prev control-3">‹</label>
    <label for="carousel-1" class="carousel-control next control-3">›</label>
    <ol class="carousel-indicators">
      <li>
        <label for="carousel-1" class="carousel-bullet">•</label>
      </li>
      <li>
        <label for="carousel-2" class="carousel-bullet">•</label>
      </li>
      <li>
        <label for="carousel-3" class="carousel-bullet">•</label>
      </li>
    </ol>
  </div>
</div>
```

完整的css：
```css
html, body {
  margin: 0px;
  padding: 0px;
  background: url("http://digital.bnint.com/filelib/s9/photos/white_wood_4500x3000_lo_res.jpg");
}

.carousel {
    position: relative;
    box-shadow: 0px 1px 6px rgba(0, 0, 0, 0.64);
    margin-top: 26px;
}

.carousel-inner {
    position: relative;
    overflow: hidden;
    width: 100%;
}

.carousel-open:checked + .carousel-item {
    position: static;
    opacity: 100;
}

.carousel-item {
    position: absolute;
    opacity: 0;
    -webkit-transition: opacity 0.6s ease-out;
    transition: opacity 0.6s ease-out;
}

.carousel-item img {
    display: block;
    height: auto;
    max-width: 100%;
}

.carousel-control {
    background: rgba(0, 0, 0, 0.28);
    border-radius: 50%;
    color: #fff;
    cursor: pointer;
    display: none;
    font-size: 40px;
    height: 40px;
    line-height: 35px;
    position: absolute;
    top: 50%;
    -webkit-transform: translate(0, -50%);
    cursor: pointer;
    -ms-transform: translate(0, -50%);
    transform: translate(0, -50%);
    text-align: center;
    width: 40px;
    z-index: 10;
}

.carousel-control.prev {
    left: 2%;
}

.carousel-control.next {
    right: 2%;
}

.carousel-control:hover {
    background: rgba(0, 0, 0, 0.8);
    color: #aaaaaa;
}

#carousel-1:checked ~ .control-1,
#carousel-2:checked ~ .control-2,
#carousel-3:checked ~ .control-3 {
    display: block;
}

.carousel-indicators {
    list-style: none;
    margin: 0;
    padding: 0;
    position: absolute;
    bottom: 2%;
    left: 0;
    right: 0;
    text-align: center;
    z-index: 10;
}

.carousel-indicators li {
    display: inline-block;
    margin: 0 5px;
}

.carousel-bullet {
    color: #fff;
    cursor: pointer;
    display: block;
    font-size: 35px;
}

.carousel-bullet:hover {
    color: #aaaaaa;
}

#carousel-1:checked ~ .control-1 ~ .carousel-indicators li:nth-child(1) .carousel-bullet,
#carousel-2:checked ~ .control-2 ~ .carousel-indicators li:nth-child(2) .carousel-bullet,
#carousel-3:checked ~ .control-3 ~ .carousel-indicators li:nth-child(3) .carousel-bullet {
    color: #428bca;
}

#title {
    width: 100%;
    position: absolute;
    padding: 0px;
    margin: 0px auto;
    text-align: center;
    font-size: 27px;
    color: rgba(255, 255, 255, 1);
    font-family: 'Open Sans', sans-serif;
    z-index: 9999;
    text-shadow: 0px 1px 2px rgba(0, 0, 0, 0.33), -1px 0px 2px rgba(255, 255, 255, 0);
}
```
原理还是 `label` 的 `for` 属性，然后用 css 做了样式以及布局。

# END
其实不用 js ，只用 css 还可以实现很多有趣的功能，比如 `tooltip`，`Toggle`，`Popover`....
更多可以看 [You-Dont-Need-Javascript](https://github.com/NamPNQ/You-Dont-Need-Javascript) 这个仓库。
