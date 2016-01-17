title: CSS 伪类before/after content内容生成技术----纯CSS实现tooltip
date: 2015-10-17 15:30:41
tags:
- 前端
- 教程
- CSS
categories: [CSS]
---
# 一、介绍before/after
CSS中的before和after伪类选择器早在CSS2时就被引入，改属性被所有主流浏览器所支持了。
before和after顾名思义，分别指的是伪元素在元素前/后添加内容，默认他们是display是inline，但是可以使用CSS设置为block。
应用before/和after也比较简单，举个例子：
```
a:after {
      content: " after ";
      display:  block;
      coloe: red;
}
```
可以在浏览器看到，a标签元素后面多出了一段文字 after。（在CSS3中伪类元素使用是如a::after的，不过目前两者并无多大区别）。
（伪元素不可通过DOM使用，IE6/7对该属性不支持）

after和before伪元素有许多用处，还可以简化代码，比如可以写一个计数器、tip小三角形、清除浮动......特别在CSS3中新加的一些属性更是丰富了它的应用，这里有个小教程，用before/after伪元素来实现一个纯CSS3的tooltip。

# 二、tooltip实现教程
这里我们主要是用草after/before伪元素content中的attr属性，先来看看实现后的样子：
## 1.实现样式
![DEMO](/img/after&before-demo.png)

## 2.代码
鼠标hover button之后，出现一个tooltip小标签。
代码先贴上：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>demo</title>
    <style>
        .btn {
            position: relative;
            display: block;
            margin: 200px auto;
            width: 200px;
            padding: 10px 20px;
            font-size: 20px;
            background: #fff;
            color: #6bdf4e;
            border: 1px solid #6bdf4e;
            cursor: pointer;
        }
        .btn::after {
            content: attr(data-tip);
            display: none;
            position: absolute;
            padding: 5px 10px;
            left: 50%;
            bottom: 100%;
            margin-bottom: 12px;
            transform: translateX(-50%);
            font-size: 16px;
            background: #000;
            color: #fff;
            cursor: default;
        }
        .btn::before {
            content: " ";
            position: absolute;
            display: none;
            left: 50%;
            bottom: 100%;
            transform: translateX(-50%);
            margin-bottom: 3px;
            width: 0;
            height: 0;
            border-left: 6px solid transparent;
            border-right: 6px solid transparent;
            border-top: 9px solid #000;
        }
        .btn:hover::after,
        .btn:hover::before {
            display: block;
        }
    </style>
</head>
<body>

    <button class="btn" data-tip="ToolTip">button</button>

</body>
</html>
```
## 3.实现过程
* 一. 创建一个标签，然后在标签内加上一个属性 data-[] = "ToolTip",[]表示的是自定义的属性名称，引号里面是tooltip需要显示的内容。
* 二. 给标签加样式，position设置为relative，因为之后伪元素需要设置绝对定位来设置位置。
* 三. 给after加样式，after是需要显示的tooltip，通过content: attr(data-tip);拿到需要显示的内容，设置绝对定位，调整位置。
* 四. 给before加样式，before需要设置成一个小三角tip放在after下面。
* 五. 给after/before的display都设置为none。
* 六. 给需要tooltip的元素伪类选择hover时设置after/before的display为block，这里需要注意的是after/before默认display为inline，所以我们前面创建调试是display应该先设置为block。
* 七. 打开浏览器查看效果~

# 三、对于不支持伪元素的IE6/7
解决方法：
```
var $beforeAfter = function(dom) {
    if (document.querySelector || !dom && dom.nodeType !== 1) return;
    var content = dom.getAttribute("data-content") || '';     
    var before = document.createElement("before")
        , after = document.createElement("after");
    before.innerHTML = content;
    after.innerHTML = content;
    dom.insertBefore(before, dom.firstChild);
    dom.appendChild(after);
};
```
# 四、tip
关于after/before还有许多有趣有用的用法，感兴趣的同学可以研究研究哦~
