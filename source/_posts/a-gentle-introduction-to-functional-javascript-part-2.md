title: JavaScript中的函数式编程二(翻译)
date: 2016-03-01 11:05:08
tags:
- JavaScript
- 前端
- 教程
- 翻译
categories: [JavaScript]
---
![js](/img/js.jpeg)

# tips
原文链接: [http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-arrays/](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-arrays/);
原文作者: [James Sinclair](http://jrsinclair.com/about.html);

# JavaScript 函数式编程
这篇文章是介绍函数式编程的四篇文章中的第二篇。在上一篇文章中，我们看到如何使用函数使代码抽象地更简洁，在这篇文章中我们将使用这个技术在列表上。

+ Part1 [组成部分和动机](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro/),
+ Part2 [使用数组和列表](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-arrays/),
+ Part3 [生成函数的函数](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/),
+ Part4 [使用函数式编程的风格](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/),

# 处理函数和集合
会想之前的文章，我们谈论了 DRY 准则。我们看到用函数绑定一系列重复的操作是很有用的。但是如果我们重复同一个函数很多次呢？就像这样：
```
function addColour(colour) {
    var rainbowEl = document.getElementById('rainbow');
    var div = document.createElement('div');
    div.style.paddingTop = '10px';
    div.style.backgroundColour = colour;
    rainbowEl.appendChild(div);
}

addColour('red');
addColour('orange');
addColour('yellow');
addColour('green');
addColour('blue');
addColour('purple');
```
在这里 addColour 这个函数被调用很多次，我们仍然重复了我们自己，这是我们一直想回避的。一种重构的方法是构建一个数组包含这些颜色的列表，然后循环调用 addColour 这个函数。
```
var colours = [
    'red', 'orange', 'yellow',
    'green', 'blue', 'purple'
];

for (var i = 0; i < colours.length; i = i + 1) {
    addColour(colours[i]);
}
```
# For-Each
JavaScript 允许我们把一个函数座位参数传递给另一个函数，编写一个 forEach 函数是相当明确了：
```
function forEach(callback, array) {
    for (var i = 0; i < array.length; i = i + 1) {
        callback(array[i], i);
    }
}
```
这个函数接受执行 callback 函数，并且把数组的每一项作为参数调用 callback 函数。
现在，在我们的例子中，我们想要对数组的每一个元素去运行 addColour 方法，使用我们的 forEach 我们仅需一行就能完美执行～
```
forEach(addColour, colours);
```
对数组中的每一个元素调用一个方法是非常有用的一个工具，JavaScript 也实现了这么一个特性组成，作为数组对象的一个方法。因此我们也可以使用它来替代我们的 forEach 方法，如下：
```
var colours = [
    'red', 'orange', 'yellow',
    'green', 'blue', 'purple'
];
colours.forEach(addColour);
```
我们可以查找更多的方法在 [MDN 的 JavaScript 参考文档里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)。

# Map
我们的 forEach 已经非常好用了，但是扔有一些局限性。如果回调函数返回一个值，那 forEach 只会忽略掉这个返回值。我们可以适当改造一下 forEach 函数，无论他返回什么类型的值我们都可以取得。我们可以得到一个数组，它包含了我们原始数字相对应的一些值。
让我们看看下面的例子，我们有一个 ID 的数组，然后我们想得到他们每个相对应的 DOM 元素集合。在程序上找到解决的方法，我们可以使用循环：
```
var ids = ['unicorn', 'fairy', 'kitten'];
var elements = [];
for (var i = 0; i < ids.length; i = i + 1) {
    elements[i] = document.getElementById(ids[i]);
}
// elements now contains the elements we are after
```
我们想要阐明计算机是如何创建一个索引变量并且增加它－－细节我们不需多考虑。让我们使用循环就像 forEach 里面一样，并且把它复制给一个叫做 map 的变量。
```
var map = function(callback, array) {
    var newArray = [];
    for (var i = 0; i < array.length; i = i + 1) {
        newArray[i] = callback(array[i], i);
    }
    return newArray;
}
```
现在我们有了一个 map 函数，可以这么实用它：
```
var getElement = function(id) {
  return document.getElementById(id);
};

var elements = map(getElement, ids);
```
这个 map 函数小而简单，但是它里面执行了另一个我们的超级函数，只需在数组的一个入口调用它一次就可以了，成倍的增长了函数的效率。
像 forEach 函数一样，JavaScript 本身也实现了 map 函数，作为数组对象的一个方法。我们可以调用这部分方法如下：
```
var ids = ['unicorn', 'fairy', 'kitten'];
var getElement = function(id) {
  return document.getElementById(id);
};
var elements = ids.map(getElement, ids);
```
[我们可以在 MDN 上查阅更多关于 map 函数的组成。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

# Reduce
嗯，map 函数事很有用的。但是我们想实现一个更有用的函数，如果我们对所有的数组元素执行函数但是返回仅仅一个值。这听起来可能有一点反常，函数返回一个值为什么会比返回多个值更有用？为了寻找答案，我们可以先看看下面这个函数是如何工作的。
为了解释，我们考虑两个相似的问题：
- 给一个数组一些数值元素，并且计算他们的和；
- 给一个数组一些字符串元素，并且用空格符在他们直接连接起来。
现在我们来看看一个愚蠢、微不足道的例子－－事实是他们也确实如此。这对于我来说相当难忍受，但是如果我们一旦学习到 reduce 函数是如何工作的，我们就可以把它应用在很多有趣的情况下。
我们再来看看程序上时如可解决这个问题的，还是循环：
```
// Given an array of numbers, calculate the sum
var numbers = [1, 3, 5, 7, 9];
var total = 0;
for (i = 0; i < numbers.length; i = i + 1) {
    total = total + numbers[i];
}
// total is 25

// Given an array of words, join them together with a space between each word.
var words = ['sparkle', 'fairies', 'are', 'amazing'];
var sentence = '';
for (i = 0; i < words.length; i++) {
    sentence = sentence + ' ' + words[i];
}
// ' sparkle fairies are amazing'
```
两个方案都是一样的其实，他们都使用for循环去迭代，他们都有个执行变量（total 和 sentence），他们都对执行变量赋值一个初始值。
让我们把他们循环中的操作抽出来写一个函数：
```
var add = function(a, b) {
    return a + b;
}

// Given an array of numbers, calculate the sum
var numbers = [1, 3, 5, 7, 9];
var total = 0;
for (i = 0; i < numbers.length; i = i + 1) {
    total = add(total, numbers[i]);
}
// total is 25

function joinWord(sentence, word) {
    return sentence + ' ' + word;
}

// Given an array of words, join them together with a space between each word.
var words = ['sparkle', 'fairies', 'are', 'amazing'];
var sentence = '';
for (i = 0; i < words.length; i++) {
    sentence = joinWord(sentence, words[i]);
}
// 'sparkle fairies are amazing'
```
现在，它变得简洁有用了，内部函数把执行变量作为他的第一个参数，然后当前遍历到的数组元素值作为第二个参数。现在我们让它更加简洁，我们把凌乱的 for 循环放到函数里面。
```
var reduce = function(callback, initialValue, array) {
    var working = initialValue;
    for (var i = 0; i < array.length; i = i + 1) {
        working = callback(working, array[i]);
    }
    return working;
};
```
现在我们又个闪闪发亮的 reduce 新函数了，让我们来使用使用它：
```
var total = reduce(add, 0, numbers);
var sentence = reduce(joinWord, '', words);
```
就像 forEach 和 map 函数，reduce 函数也是 JavaScript 数组对象的标准方法之一。
我们可以这么使用它：
```
var total = numbers.reduce(add, 0);
var sentence = words.reduce(joinWord, '');
```
[我们可以在 MDN 上查阅更多关于 reduce 函数的组成。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

# 总结
正如我们前面所提及，他们都是很简单的例子－－add 和 joinWord 函数都非常地简单。更小，更简单的函数易于思考和测试。当我们把两个小而简单的函数结合起来的时候（就像 add 和 reduce），他的结果比起编写一个大而复杂的函数来说仍然很好解释。
但是，正如我前面所说，我们可以把很多方法结合起来做一些更有意思的事情。
让我们尝试更复杂一些，我们把一个数据对象用 map 和 reduce 函数转化成一个 HTML 列表，数据如下：
```
var ponies = [
    [
        ['name', 'Fluttershy'],
        ['image', 'http://tinyurl.com/gpbnlf6'],
        ['description', 'Fluttershy is a female Pegasus pony and one of the main characters of My Little Pony Friendship is Magic.']
    ],
    [
        ['name', 'Applejack'],
        ['image', 'http://tinyurl.com/gkur8a6'],
        ['description', 'Applejack is a female Earth pony and one of the main characters of My Little Pony Friendship is Magic.']
    ],
    [
        ['name', 'Twilight Sparkle'],
        ['image', 'http://tinyurl.com/hj877vs'],
        ['description', 'Twilight Sparkle is the primary main character of My Little Pony Friendship is Magic.']
    ]
];
```
这些数据不是十分整洁，如果把里面的数组变成对象会更清晰。不过，在之前我们可以 reduce 函数去估算一些简单的值就像数字或者字符串，但是没人敢说 reduce 返回的值一定简单。我们可以把它使用在 对象，数组 或者甚至是 DOM 元素 上面。让我们创造一个函数把内部的数组（像 ['name', 'Fluttershy']）以键值对的形式添加到一个对象之中。
```
var addToObject = function(obj, arr) {
    obj[arr[0]] = arr[1];
    return obj;
};
```
使用 addToObject 函数，我们可以把每个数组转化成对象：
```
var ponyArrayToObject = function(ponyArray) {
    return reduce(addToObject, {}, ponyArray);
};
```
然后使用 map 函数把整个数组转化的更简洁清晰：
```
var tidyPonies = map(ponyArrayToObject, ponies);
```
现在我们有了一个对象组成的数组，可以从[Thomas Fuchs’ tweet-sized templating engine](http://mir.aculo.us/2011/03/09/little-helpers-a-tweet-sized-javascript-templating-engine/)获得一些帮助，我们可以再使用 reduce 函数把它转换成 HTML 片段。模版函数接受一个模版字符串和一个对象，他会把字符串当中 mustache-wrapped 格式的部分（像, {name} 或者 {image})）用对象中包含的属性值类替换。比如：
```
var data = { name: "Fluttershy" };
t("Hello {name}!", data);
// "Hello Fluttershy!"

data = { who: "Fluttershy", time: Date.now() };
t("Hello {name}! It's {time} ms since epoch.", data);
// "Hello Fluttershy! It's 1454135887369 ms since epoch."
```
所以，如果我们想要吧对象转换成列表项，我们可以这么做：
```
var ponyToListItem = function(pony) {
    var template = '<li><img src="{image}" alt="{name}"/>' +
                   '<div><h3>{name}</h3><p>{description}</p>' +
                   '</div></li>';
    return t(template, pony);
};
```
在上面我们把对象转换成了 html 片段，但是如果想转换整个数组，我们就需要 reduce 和 joinWord 方法：
```
var ponyList = map(ponyToListItem, tidyPonies);
var html = '<ul>' + reduce(joinWord, '', ponyList) + '</ul>';
```

我们可以在[http://jsbin.com/wuzini/edit?html,js,output](http://jsbin.com/wuzini/edit?html,js,output)看到整个运行结果。

当你理解 reduce 和 map 方法之后，你就不再需要老旧的 for 循环了。事实上，如果你决定在你的下一个项目上完全不使用 for 循环将会是一个很有意思的挑战。当你使用 reduce 和 map 越来越多的时候，你就会注意还有没有更多的部分可以被抽象出来。一些常见的包括[过滤filtering](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)，或者[plucking](http://ramdajs.com/docs/#pluck)。这些部分被使用的越来越频繁，人们把他们放到一个函数式编程的库里面，有一些流行的库包括：
- [Ramda](http://ramdajs.com/0.19.1/index.html),
- [Lodash](https://lodash.com/), and
- [Underscore](http://underscorejs.org/).


－－－－－－－－－－－－－－－－－

<br/>
未亡待续...
[阅读下一节～]
