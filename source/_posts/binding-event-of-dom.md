title: 从一个事件绑定说起 - DOM
date: 2016-08-15 10:45:32
tags:
- JavaScript
categories: [JavaScript]
bgimage: http://fakeimg.pl/2000x800/ff6a6f/fff/?text=EVENT
---

# 事件绑定的方式
给 DOM 元素绑定事件分为两大类：在 html 中直接绑定 和 在 JavaScript 中绑定。

# Bind in HTML
在 HTML 中绑定事件叫做内联绑定事件，HTML 的元素中有如 `onclick` 这样的 `on***` 属性，它可以给这个 DOM 元素绑定一个类型的事件，主要是这样的：

```
<div onclick="***">CLICK ME</div>
```

这里的 `***` 有两种形式：

- 用字符串表示一段函数：

```html
<div onclick="var a = 1; console.log(a); console.log(this);">CLICK ME</div>
```

点击可以发现，console:

```
1
<div onclick=​"var a = 1;​ console.log(a)​;​ console.log(this)​;​">​CLICK ME​</div>​
```

`var a = 1; console.log(a); console.log(this);` 这一段字符串被当作 js 执行了，同时 `this` 指向当前这个点击的元素。

- 用函数名表示：

```html
<div onclick="foo(this)">CLICK ME</div>
```

这里需要添加 `script` 去定义函数：

```html
<script>
  function foo(_this) {
    console.log(_this);
    console.log(this);
  }
</script>
```

可以观察到，console：

```
<div onclick=​"var a = 1;​ console.log(a)​;​ console.log(this)​;​">​CLICK ME​</div>​
window
```

这里的 `this` 指向了全局，传入的参数是点击的 DOM 元素，其实和第一种方式是一样的，都是在 `onclick=` 后面指向了一段js的字符串，不同的是在这个字符串中是执行了一个函数名，而这个函数我们在全局中定义了，所以点击的时候可以执行，然后传入的参数 `this` 也就是一样的道理了。

或者多说一句，这里的字符串才是真正赋值给 `onclick` 的函数，这里我们是在函数里面再执行了 `foo` 函数。

**然而这内联的方式绑定时间不利于分离，所以一般我们不推荐这种做法，所以也就不再多阐述了**

# Bind in JavaScript

## dom.onclick
先上栗子：

```html
<div id="clickme">CLICK ME</div>
```

```js
document.getElementById('clickme').onclick = function (e) {
  console.log(this.id);
  console.log(e);
};
```

观察 console：

```
clickme
MouseEvent {isTrusted: true, screenX: 65, screenY: 87, clientX: 65, clientY: 13…}
```

首先，我们获取到了 `dom` 元素，然后给它的 `onclick` 属性赋值了一个函数；

点击 `dom` 我们发现那个函数执行了，同时发现函数中的 `this` 是指向当前的这个 `dom` 元素；

细细一想，其实这和前面用的在 `html` 中 ***内联*** 绑定函数是一样的，我们同样是给 `dom` 的 `onclick` 属性赋值一个函数，然后函数中的 `this` 指向当前元素，只是这个过程这里我们是在 js 中做的；

而还有一点区别就是前面的是赋值一段 js 字符串，这里是赋值一个函数，所以可以接受一个参数：`event`，这个参数是点击的事件对象。
<br>

用赋值绑定函数的一个缺点就是它只能绑定一次：

```js
document.getElementById('clickme').onclick = function (e) {
  console.log(this.id);
};
document.getElementById('clickme').onclick = function (e) {
  console.log(this.id);
};
```

可以看到这里只打印了一次 `clickme`。


## addEventListener
**这个才是我们需要重点介绍的一个函数**

### 语法

```js
target.addEventListener(type, listener[, useCapture]);
```

- `target` : 表示要监听事件的目标对象，可以是一个文档上的元素 `Document` `本身，Window` 或者 `XMLHttpRequest`；
- `type` : 表示事件类型的字符串，比如: `click`、`change`、`touchstart` ...；
- `listener` : 当指定的事件类型发生时被通知到的一个对象。该参数必是实现 EventListener 接口的一个对象或函数。
- `useCapture` : 设置事件的 **捕获或者冒泡** (后文阐述) ，`true`: 捕获，`false`: 冒泡*，默认为 `false`。

### 简单栗子

```html
<div id="clickme">CLICK ME</div>
```

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1);

function foo1(event) {
  console.log(this.id);
  console.log(event);
}
```

观察 console：

```
clickme
MouseEvent {isTrusted: true, screenX: 37, screenY: 88, clientX: 37, clientY: 14…}
```

监听函数中的 `this` 指向当前的 `dom` 元素，函数接受一个 `event` 参数。

### 绑定函数

#### 绑定多个函数
`addEventListener` 可以给同一个 `dom` 元素绑定多个函数：

```html
<div id="clickme">CLICK ME</div>
```

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1);
clickme.addEventListener('click', foo2);

function foo1(event) {
  console.log(111);
}

function foo1(event) {
  console.log(222);
}
```

可以看到 console：

```
111
222
```

我们可以看到两个函数都执行了，并且执行顺序按照绑定的顺序执行。

***改变一下***，如果我们的 `useCapture` 参数不同呢？
看下面 3 组对比：

##### 1

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1, true);
clickme.addEventListener('click', foo2, true);

function foo1(event) {
  console.log(111);
}

function foo1(event) {
  console.log(222);
}
```

##### 2

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1, true);
clickme.addEventListener('click', foo2, false);

function foo1(event) {
  console.log(111);
}

function foo1(event) {
  console.log(222);
}
```

##### 3

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1, false);
clickme.addEventListener('click', foo2, true);

function foo1(event) {
  console.log(111);
}

function foo1(event) {
  console.log(222);
}
```

可以看到，console 并没有改变，所以执行顺序只和绑定顺序有关，和 `useCapture` 无关。

结论：
**我们可以给一个 `dom` 元素绑定多个函数，并且它的执行顺序按照绑定的顺序执行。**


#### 同一个元素绑定同一个函数
我们给一个 `dom` 元素绑定同一个函数两次试试：

```html
<div id="clickme">CLICK ME</div>
```

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1);
clickme.addEventListener('click', foo1);

function foo1(event) {
  console.log(111);
}
```

观察 console:
```
111
```

可以看到函数只执行了一次；

换一种方式：

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1, true);
clickme.addEventListener('click', foo1, false);

function foo1(event) {
  console.log(111);
}
```

观察 console:
```
111
111
```

可以看到函数执行了两次。

结论：
**我们可以给一个 `dom` 元素绑定同一个函数，最多只能绑定 `useCapture` 类型不同的两次。**

### IE下使用attachEvent/detachEvent

`addEventListener` 只支持到 IE 9，所以为了兼容性考虑，在兼容 IE 8 以及以下浏览器可以用 `attachEvent` 函数，和 `addEventListener` 函数表现一样，除了它绑定函数的 `this` 会指向全局这个缺点以外

# 事件执行顺序的 PK
## 1. addEventListener 和 dom.onclick

```html
<div id="clickme">CLICK ME</div>
```

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1, true);
clickme.onclick = foo2;
clickme.addEventListener('click', foo3, true);

function foo1(event) {
  console.log(111);
}

function foo2(event) {
  console.log(222);
}

function foo3(event) {
  console.log(333);
}
```

观察 console：

```
111
222
333
```

**可见执行顺序只和绑定顺序有关**

## 2. addEventListener 间的比较
[见上文](#绑定多个函数)

# 事件的解绑
与事件绑定相对应的就是事件解绑了。

## 1. 通过 dom 的 on** 属性设置的事件
**对于 Bind in HTML 和 `dom.onclick` 绑定的事件都可以用 `dom.onclick = null` 来解绑事件。**

## 2. removeEventListener
通过 `addEventListener` 绑定的事件可以使用 `removeEventListener` 来解绑， `removeEventListener` 接受的参数和 `addEventListener` 是一样的

```HTML
<div id="clickme">CLICK ME</div>
```

```js
var clickme = document.getElementById('clickme');

clickme.addEventListener('click', foo1);

function foo1(event) {
  console.log(111);
}
clickme.removeEventListener('click', foo1, true);

```

这里发现事件并没有取消绑定，发现 `removeEventListener` 的 `useCapture` 的参数原来和绑定时候传入的不一致，我们改成 `false` 之后发现事件取消了。

结论：
**对于使用 `removeEventListener` 函数解绑事件，需要传入的 `listener` `useCapture` 应和 `addEventListener` 一致才可以解绑事件。**

## 3. detachEvent
与 `attachEvent` 对应



# DOM 事件
![DEMO](/img/dom-event.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div class="div1">
    <div class="div2">
      <div class="div3">

      </div>
    </div>
  </div>
</body>
</html>
```

## 事件冒泡
事件开始时由最具体的元素接受，然后逐级向上传播到较为不具体的节点。

比如上面的 HTML ，冒泡的顺序： div3 -> div3 -> div1 -> body -> html -> document (-> window)

## 事件捕获
事件捕获的思想是不太具体的DOM节点应该更早接收到事件，而最具体的节点应该最后接收到事件。与事件冒泡的顺序相反。

比如上面的 HTML ，捕获的顺序： document -> html -> body -> div1 -> div2 -> div3

## DOM事件流
DOM事件流包括三个阶段：事件捕获阶段、处于目标阶段、事件冒泡阶段。首先发生的事件捕获，为截获事件提供机会。然后是实际的目标接受事件。最后一个阶段是时间冒泡阶段，可以在这个阶段对事件做出响应。

## 回到 addEventListener
我们来做几个小对比：

```html
<div id="wrap">
  <div id="clickme">CLICK ME</div>
</div>
```

### 栗子 1

```js

document.getElementById('wrap').addEventListener('click', foo1, false);
document.getElementById('clickme').addEventListener('click', foo2, false);

function foo1(event) {
  console.log(111);
}
function foo2(event) {
  console.log(222);
}

```

console：

```
222
111
```

这里两个事件都是冒泡类型，所以是从内到外；


### 栗子 2

```js

document.getElementById('wrap').addEventListener('click', foo1, true);
document.getElementById('clickme').addEventListener('click', foo2, true);

function foo1(event) {
  console.log(111);
}
function foo2(event) {
  console.log(222);
}

```

console：

```
111
222
```

这里两个事件都是捕获类型，所以是从外到内；

### 栗子 3

```js

document.getElementById('wrap').addEventListener('click', foo1, false);
document.getElementById('clickme').addEventListener('click', foo2, true);

function foo1(event) {
  console.log(111);
}
function foo2(event) {
  console.log(222);
}

```

console：

```
222
111
```

wrap 事件是冒泡，clickme 事件是捕获，根据 dom 的事件流，先执行了捕获阶段（这里是目标阶段）再到冒泡阶段。

### 栗子 4

```js

document.getElementById('wrap').addEventListener('click', foo1, true);
document.getElementById('clickme').addEventListener('click', foo2, false);

function foo1(event) {
  console.log(111);
}
function foo2(event) {
  console.log(222);
}

```

console：

```
111
222
```

clickme 事件是冒泡，wrap 事件是捕获，根据 dom 的事件流，先执行了捕获阶段（这里是目标阶段）再到冒泡阶段。


## 阻止 冒泡／捕获

```html
<div id="wrap">
  <div id="clickme">CLICK ME</div>
</div>
```

```js

document.getElementById('wrap').addEventListener('click', foo1);
document.getElementById('clickme').addEventListener('click', foo2);

function foo1(event) {
  console.log(111);
}
function foo2(event) {
  event.stopPropagation();
  console.log(222);
}
```

console：

```
222
```

这里我们在 `clickme` 的目标阶段阻止了事件流，所以事件不会继续冒泡，所以 `wrap` 的冒泡事件不会执行。

tip:
IE8以及以前可以通过 `window.event.cancelBubble=true` 来阻止事件流。
