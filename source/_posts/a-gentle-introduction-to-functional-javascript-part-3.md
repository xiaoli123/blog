title: JavaScript中的函数式编程三(翻译)
date: 2016-03-13 20:30:08
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
这篇文章是介绍javascript函数式编程系列中的第四篇文章，在上一篇文章中我们学习到了如何在数组和列表中使用函数式编程，在这篇文章中，我们研究一些可以创造函数的高阶函数。

+ Part1 [组成部分和动机](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro/),
+ Part2 [使用数组和列表](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-arrays/),
+ Part3 [生成函数的函数](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/),
+ Part4 [使用函数式编程的风格](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-functions/),

# 创建函数的函数
（真拗口－默默吐槽）
在上一篇文章的末尾，我说过仅仅把处理一系列过程到函数并不适用于所有情况，因为如果只是创造一个执行一系列过程的函数有时候会让我们的代码失控起来。我的意思是说我吗一开始抽象收集了一些命令道一个函数中，然后抽象出for循环到``map``和``reduce``中，接下来我们应该做的是抽象重构一部分来创造函数。我们开始使用函数去创造函数，这将会非常有用并且我们的代码也会更加幽雅，但是这看起来可能会和你平常写的javascript有点不同。

# 更多的创建块
能够创建函数的函数通常称为高阶函数，为了去理解它，我看需要重新看看javascript的一些特性，它们让高阶函数成为可能。

## 闭包和作用域
javascript中让人很头疼的特性中有一个是一个函数可以被“看见”。在javascript中，如果你定义了一个变量在一个函数中，在这个函数的外部就不能“看见”这一个变量，比如：
```
var thing = 'bat';

var sing = function() {
    // This function can 'see' thing
    var line = 'Twinkle, twinkle, little ' + thing;
    log(line);
};

sing();
// Twinkle, twinkle, little bat

// Outside the function we can't see message though
log(line);
// undefined
```
然而，如果你在一个函数中定义课一个函数，里面的函数可以“看见”外面的这个函数里面的变量：
```
var outer = function() {
    var outerVar = 'Hatter';
    var inner = function() {
         // We can 'see' outerVar here
         console.log(outerVar);
         // Hatter

         var innerVar = 'Dormouse';
         // innerVar is only visible here inside inner()
    }

    // innerVar is not visible here.
}
```
这个特性我们习以为常，但是如果我们把变量作为参数传递进来，我们就很难去梳理那个函数可以“看见”哪个变量。他然我们一开始很困扰，但是请耐心：让我们看看你在哪里定义的函数，再找出哪些变量是在那个地方可以访问到的。它们可能并不会如你预期的那个在调用函数的那个地方。
## 特别的参数变量
在你创建一个函数的同时，它同样创建了一个特别的参数－－``arguments``，它就像一个数组。它包含了被传到这个函数的参数，比如：
```
var showArgs = function(a, b) {
    console.log(arguments);
}
showArgs('Tweedledee', 'Tweedledum');
//=> { '0': 'Tweedledee', '1': 'Tweedledum' }
```
注意输出，它更像一个key都是整数的对象而不是一个数组，事实确实如此。
有趣的是``arguments``包含了所有被传到这个函数的参数而不管函数的参数被定义了几个。所以，如果你调用一个函数并且传递给他额外的参数，它们都可以在``arguments``里面的到：
```
showArgs('a', 'l', 'i', 'c', 'e');
//=> { '0': 'a', '1': 'l', '2': 'i', '3': 'c', '4': 'e'
```
``arguments``已有``length``，就像数组一样：
```
var argsLen = function() {
    console.log(arguments.length);
}
argsLen('a', 'l', 'i', 'c', 'e');
//=> 5
```
让``arguments``像是数组一样访问是很有用的，饿哦吗可以把``arguments``转化成一个真正的数组通过``call``方法：
```
var showArgsAsArray = function() {
    var args = Array.prototype.slice.call(arguments, 0);
    console.log(args);
}
showArgsAsArray('Tweedledee', 'Tweedledum');
//=> [ 'Tweedledee', 'Tweedledum' ]
```
这个``arguments``变量在创造函数中可以让我们得到传递进来的参数的数量，我们离的更近了。

## call 和 apply
在之前的数组中，我们看到了他有``reduce``和``map``这样的方法，函数自然也有他的方法。
普通的方法调用函数是在函数后面写一个小括号，然后把参数写在里面：
```
function twinkleTwinkle(thing) {
    console.log('Twinkle, twinkle, little ' + thing);
}
twinkleTwinkle('bat');
//=> Twinkle, twinkle, little bat
```
还没翻译完/.....

－－－－－－－－－－－－－－－－－

<br/>
未亡待续...
[阅读下一节～]
