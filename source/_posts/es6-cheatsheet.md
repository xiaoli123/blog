title: ES6 手册
date: 2016-07-27 09:05:05
tags:
- JavaScript

categories: [JavaScript]
bgimage: /img/es6-cheatsheet.jpg
---
# tip
文章是翻译，主要是很多 ES6 的用法技巧以及最佳实践～
原文 [https://github.com/DrkSephy/es6-cheatsheet](https://github.com/DrkSephy/es6-cheatsheet)

------
![es6](/img/es6.jpg)

# ES6 手册
这篇手册包含了 ES2015(ES6) 的使用小技巧、最佳实践以及可以给你每天的工作参考的代码片段。

## 内容列表
- [var 和 let/const 的比较](#var_和_let/const_的比较)
- [用块级作用域代替 IIFES](#用块级作用域代替_IIFES)
- [箭头函数](#箭头函数)
- [字符串](#字符串)
- [解构](#解构)
- [模块](#模块)
- [参数](#参数)
- [类 Classes](#类_Classes)
- [Symbols](#Symbols)
- [Maps](#Maps)
- [WeakMaps](#WeakMaps)
- [Promises](#Promises)
- [Generators 生成器](#Generators_生成器)
- [Async Await](#Async_Await)
- [Getter/Setter 函数](#Getter/Setter_函数)

## var 和 let/const 的比较
> 除了 ``var`` ，我们现在还可以使用两个新的标示符来定义一个变量 —— ``let`` 和 ``const``。和 ``var`` 不一样的是，``let`` 和 ``const`` 不存在变量提升。

使用 ``var`` 的栗子：
```
var snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        var snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // undefined
```

当我们用 ``let`` 代替 ``var`` 的时候，观察会发生什么：
```
let snack = 'Meow Mix';

function getFood(food) {
    if (food) {
        let snack = 'Friskies';
        return snack;
    }
    return snack;
}

getFood(false); // 'Meow Mix'
```

当我们重构使用 ``var`` 的老代码的时候应该注意上面的变化。盲目地使用 ``let`` 替换 ``var`` 可能会出现出乎意料的情况。

> **注意**： ``let`` 和 ``const`` 是块级作用域，因此在变量未被定义之前使用它会产生一个 ``ReferenceError``。

```
console.log(x); // ReferenceError: x is not defined

let x = 'hi';
```

> **最佳实践**： 在遗留代码中放弃使用 ``var`` 声明意味着需要很小心地重构；在新的项目代码中，使用 ``let`` 声明一个可以改变的变量，用 ``const`` 声明一个不能被重新赋值的变量。


## 用块级作用域代替 IIFES
> **函数立即执行表达式** 的通常用途是创造一个内部的作用域，在 ES6 中，我们能够创造一个块级作用域而不仅限于函数作用域：

IIFES：
```
(function () {
    var food = 'Meow Mix';
}());

console.log(food); // Reference Error
```

使用 ES6 的块级作用域：
```
{
    let food = 'Meow Mix';
}

console.log(food); // Reference Error
```

## 箭头函数
我们经常需要给回调函数维护一个词法作用域的上下文 ``this``。
看看这个栗子：
```
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character; // Cannot read property 'name' of undefined
    });
};
```

一个常用的解决办法是把 ``this`` 存在一个变量中：
```
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    var that = this; // Store the context of this
    return arr.map(function (character) {
        return that.name + character;
    });
};
```

我们也可以传递一个合适的 ``this`` 上下文：
```
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }, this);
}
```

我们还可以绑定上下文：
```
function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(function (character) {
        return this.name + character;
    }.bind(this));
};
```

使用 **箭头函数**，``this`` 将不会受到影响，并且我们可以重写上面的函数：
```

function Person(name) {
    this.name = name;
}

Person.prototype.prefixName = function (arr) {
    return arr.map(character => this.name + character);
};
```

> **最佳实践**：当你需要维护一个 ``this`` 上下文的时候使用 **箭头函数**。

在我们写一个函数的时候，箭头函数更加简洁并且可以很简单地返回一个值：
```
var squares = arr.map(function (x) { return x * x }); // Function Expression
```

```
const arr = [1, 2, 3, 4, 5];
const squares = arr.map(x => x * x); // Arrow Function for terser implementation
```

> **最佳实践**：尽可能使用箭头函数代替原来的写法。

## 字符串
在 ES6 中，标准库升级了很多，在这些变化中有许多新的对于字符串的函数，比如 ``.includes()`` 和 ``.repeat()``。

### .includes( )
```
var string = 'food';
var substring = 'foo';

console.log(string.indexOf(substring) > -1);
```

之前我们使用 ``indexOf()`` 函数的返回值是否 ``>-1`` 来判断字符串是否包含某些字符串，现在我们更简单地使用 ``.includes()`` 来返回一个布尔值来判断：
```
const string = 'food';
const substring = 'foo';

console.log(string.includes(substring)); // true
```

### .repeat( )
```
function repeat(string, count) {
    var strings = [];
    while(strings.length < count) {
        strings.push(string);
    }
    return strings.join('');
}
```

在 ES6 中，可以更简便地实现：
```
// String.repeat(numberOfRepetitions)
'meow'.repeat(3); // 'meowmeowmeow'
```

### 模版字符串
使用 **模版字符串** 我们就可以不用对某些特殊自负进行转义处理了：
```
var text = "This string contains \"double quotes\" which are escaped.";
```
```
let text = `This string contains "double quotes" which don't need to be escaped anymore.`;
```

**模版字符串** 还支持插入，可以把变量值和字符串连接起来.
```
var name = 'Tiger';
var age = 13;

console.log('My cat is named ' + name + ' and is ' + age + ' years old.');
```

更简单：
```
const name = 'Tiger';
const age = 13;

console.log(`My cat is named ${name} and is ${age} years old.`);
```

在 ES5 中，需要换行时，需要这样：
```
var text = (
    'cat\n' +
    'dog\n' +
    'nickelodeon'
);
```

或者这样：
```
var text = [
    'cat',
    'dog',
    'nickelodeon'
].join('\n');
```

**模版字符串** 可以支持换行并且不需要额外的处理：
```
let text = ( `cat
dog
nickelodeon`
);
```

**模版字符串** 还支持表达式：
```
let today = new Date();
let text = `The time and date is ${today.toLocaleString()}`;
```

## 解构
结构可以让我们用一个更简便的语法从一个数组或者对象（即使是深层的）中分离出来值，并存储他们。

### 结构数组
```
var arr = [1, 2, 3, 4];
var a = arr[0];
var b = arr[1];
var c = arr[2];
var d = arr[3];
```
```
let [a, b, c, d] = [1, 2, 3, 4];

console.log(a); // 1
console.log(b); // 2
```

### 结构对象
```
var luke = { occupation: 'jedi', father: 'anakin' };
var occupation = luke.occupation; // 'jedi'
var father = luke.father; // 'anakin'
```
```
let luke = { occupation: 'jedi', father: 'anakin' };
let {occupation, father} = luke;

console.log(occupation); // 'jedi'
console.log(father); // 'anakin'
```

## 模块
在 ES6 之前，我们使用 ``Browserify`` 这样的库来创建客户端的模块化，在 ``node.js`` 中使用 ``require ``。
在 ES6 中，我们可以直接使用所有类型的模块化（AMD 和 CommonJS）。

### 使用 CommonJS 的出口
```
module.exports = 1;
module.exports = { foo: 'bar' };
module.exports = ['foo', 'bar'];
module.exports = function bar () {};
```

### 使用 ES6 的出口
在 ES6 中我们可以暴漏多个值，使用 ``Exports``：
```
export let name = 'David';
export let age  = 25;​​
```

或者暴露一个对象列表：
```
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

export { sumTwo, sumThree };
```

我们还可以暴露函数、对象和其他的值，通过简单地使用 ``export`` 这个关键字：
```
export function sumTwo(a, b) {
    return a + b;
}

export function sumThree(a, b, c) {
    return a + b + c;
}
```

最后，我们还可以绑定一个默认的输出：
```
function sumTwo(a, b) {
    return a + b;
}

function sumThree(a, b, c) {
    return a + b + c;
}

let api = {
    sumTwo,
    sumThree
};

export default api;

/* Which is the same as
 * export { api as default };
 */
```

> **最佳实践**：总是在模块的最后面使用 ``export default`` 方法，这可以让暴露的东西更加清晰并且可以节省时间去找出暴漏出来值的名字。尤其如此，在 CommonJS 中通常的实践就是暴露一个简单的值或者对象。坚持这种模式，可以让我们的代码更加可读，并且在 ES6 和 CommonJS 模块之间更好地兼容。

### ES6 中的导入
在 ES6 中同样提供了多样的导入方式，我们可以这么导入一个整个文件：
```
import 'underscore';
```

> 需要着重注意的一点是简单的导入整个文件会在那个文件的顶部执行所有的代码

和 Python 中类似，我们可以命名导入的值：
```
import { sumTwo, sumThree } from 'math/addition';
```

我们还可以重命名导入：
```
import {
    sumTwo as addTwoNumbers,
    sumThree as sumThreeNumbers
} from 'math/addition';
```

另外，我们可以导入所有的东西（整体加载）：
```
import * as util from 'math/addition';
```

最后，我们可以从一个模块中导入一个值的列表：
```
import * as additionUtil from 'math/addition';
const { sumTwo, sumThree } = additionUtil;
```

可以像这样导入默认绑定的输出：
```
import api from 'math/addition';
// Same as: import { default as api } from 'math/addition';
```

虽然最好保持出口的简单，但如果需要的话我们有时可以混合默认的进口和混合进口。当我们这样出口的时候：
```
// foos.js
export { foo as default, foo1, foo2 };
```
我们可以这样导入它们：
```
import foo, { foo1, foo2 } from 'foos';
```

当我们用 commonjs 的语法导入一个模块的出口时（比如 React），我们可以这样做：
```
import React from 'react';
const { Component, PropTypes } = React;
```
还有更精简的写法：
```
import React, { Component, PropTypes } from 'react';
```

> **注意**：导出的值是动态引用的，而不是拷贝。因此，在一个模块中改变一个变量的绑定将影响输出模块中的值。应该避免改变这些导出值的公共接口。（原文这里我觉得有误）


## 参数
在 ES5 中，在函数中我们需要各种操作去处理 ***默认参数***、***不定参数*** 和 ***重命名参数*** 等需求，在 ES6 中我们可以使用更简洁的语法完成这些需求：

### 默认参数
```
function addTwoNumbers(x, y) {
    x = x || 0;
    y = y || 0;
    return x + y;
}
```
ES6 中，函数的参数可以支持设置默认值：
```
function addTwoNumbers(x=0, y=0) {
    return x + y;
}
```
```
addTwoNumbers(2, 4); // 6
addTwoNumbers(2); // 2
addTwoNumbers(); // 0
```

### rest 参数
在 ES5 中，我们需要这么处理不定参数：
```
function logArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}
```

使用 **rest** ，我们就可以处理不确定数目的参数：
```
function logArguments(...args) {
    for (let arg of args) {
        console.log(arg);
    }
}
```

### 命名参数
在 ES5 中是使用配置对象的模式来处理命名参数，jQuery 中的使用：
```
function initializeCanvas(options) {
    var height = options.height || 600;
    var width  = options.width  || 400;
    var lineStroke = options.lineStroke || 'black';
}
```

我们可以利用解构的一个函数的形参实现相同的功能：
```
function initializeCanvas(
    { height=600, width=400, lineStroke='black'}) {
        // Use variables height, width, lineStroke here
    }
```

如果我们想使整个值可选择，我们可以结构将一个空的对象：
```
function initializeCanvas(
    { height=600, width=400, lineStroke='black'} = {}) {
        // ...
    }
```

### 展开操作
在 ES5 中，我们可以 ``apply`` ``Math.max`` 方法来获得一个数组中的最大值：
```
Math.max.apply(null, [-1, 100, 9001, -32]); // 9001
```

在 ES6 中，我们可以通过展开操作把一个数组的值作为参数传递给一个函数：
```
Math.max(...[-1, 100, 9001, -32]); // 9001
```

我们可以更简洁地使用这个语法来合并数组：
```
let cities = ['San Francisco', 'Los Angeles'];
let places = ['Miami', ...cities, 'Chicago']; // ['Miami', 'San Francisco', 'Los Angeles', 'Chicago']
```

## 类 Classes
在 ES6 之前，我们通过构造函数来创造一个类，并且通过原型来扩展属性：
```
function Person(name, age, gender) {
    this.name   = name;
    this.age    = age;
    this.gender = gender;
}

Person.prototype.incrementAge = function () {
    return this.age += 1;
};
```
然后可以这样继承类：
```
function Personal(name, age, gender, occupation, hobby) {
    Person.call(this, name, age, gender);
    this.occupation = occupation;
    this.hobby = hobby;
}

Personal.prototype = Object.create(Person.prototype);
Personal.prototype.constructor = Personal;
Personal.prototype.incrementAge = function () {
    Person.prototype.incrementAge.call(this);
    this.age += 20;
    console.log(this.age);
};
```

在 ES6 中，提供了更多的语法糖，可以直接创造一个类：
```
class Person {
    constructor(name, age, gender) {
        this.name   = name;
        this.age    = age;
        this.gender = gender;
    }

    incrementAge() {
      this.age += 1;
    }
}
```
使用 ``extends`` 关键字来继承一个类：
```
class Personal extends Person {
    constructor(name, age, gender, occupation, hobby) {
        super(name, age, gender);
        this.occupation = occupation;
        this.hobby = hobby;
    }

    incrementAge() {
        super.incrementAge();
        this.age += 20;
        console.log(this.age);
    }
}
```

> **最佳实践**：虽然使用 ES6 的语法创造类的时候，js引擎是如何实现类以及如何操作原型是令人费解的，但是未来对初学者来说这是一个好的开始，同时也可以让我们写更简洁的代码。


## Symbols
``Symbols`` 在 ES6 之前就已经存在，但是我们现在可以直接使用一个开发的接口了。``Symbols`` 是不可改变并且是第一无二的，可以在任意哈希中作一个key。

### Symbol()
调用 ``Symbol()`` 或者 ``Symbol(description)`` 可以创造一个第一无二的符号，但是在全局是看不到的。``Symbol()`` 的一个使用情况是给一个类或者命名空间打上补丁，但是可以确定的是你不会去更新它。比如，你想给 ``React.Component`` 类添加一个 ``refreshComponent`` 方法，但是可以确定的是你不会在之后更新这个方法：
```
const refreshComponent = Symbol();

React.Component.prototype[refreshComponent] = () => {
    // do something
}
```

### Symbol.for(key)
``Symbol.for(key)`` 同样会创造一个独一无二并且不可改变的 Symbol，但是它可以全局看到，两个相同的调用 ``Symbol.for(key)`` 会返回同一个 ``Symbol`` 类：
```
Symbol('foo') === Symbol('foo') // false
Symbol.for('foo') === Symbol('foo') // false
Symbol.for('foo') === Symbol.for('foo') // true
```

对于 Symbols 的普遍用法（尤其是``Symbol.for(key)``）是为了协同性。它可以通过在一个第三方插件中已知的接口中对象中的参数中寻找用 Symbol 成员来实现，比如：
```
function reader(obj) {
    const specialRead = Symbol.for('specialRead');
    if (obj[specialRead]) {
        const reader = obj[specialRead]();
        // do something with reader
    } else {
        throw new TypeError('object cannot be read');
    }
}
```
在另一个库中：
```
const specialRead = Symbol.for('specialRead');

class SomeReadableType {
    [specialRead]() {
        const reader = createSomeReaderFrom(this);
        return reader;
    }
}
```


## Maps
Maps 在 JavaScript 中是一个非常必须的数据结构，在 ES6 之前，我们通过对象来实现哈希表：
```
var map = new Object();
map[key1] = 'value1';
map[key2] = 'value2';
```

但是它并不能防止我们偶然地用一些特殊的属性名重写函数：
```
> getOwnProperty({ hasOwnProperty: 'Hah, overwritten'}, 'Pwned');
> TypeError: Property 'hasOwnProperty' is not a function
```

实际上 **Maps** 允许我们对值进行 `set`、`get` 和 `search` 操作：
```
let map = new Map();
> map.set('name', 'david');
> map.get('name'); // david
> map.has('name'); // true
```

**Maps** 更令人惊奇的部分就是它不仅限于使用字符串作为 key，还可以用其他任何类型的数据作为 key：
```
let map = new Map([
    ['name', 'david'],
    [true, 'false'],
    [1, 'one'],
    [{}, 'object'],
    [function () {}, 'function']
]);

for (let key of map.keys()) {
    console.log(typeof key);
    // > string, boolean, number, object, function
}
```

> **注意**：但我们使用 ``map.get()`` 方法去测试相等时，如果在 Maps 中使用 函数 或者 对象 等非原始类型值的时候测试将不起作用，所以我们应该使用 Strings, Booleans 和 Numbers 这样的原始类型的值。

我们还可以使用 ``.entries()`` 来遍历迭代：
```
for (let [key, value] of map.entries()) {
    console.log(key, value);
}
```


## WeakMaps
在 ES6 之前，为了存储私有变量，我们有各种各样的方法去实现，其中一种方法就是用命名约定：
```
class Person {
    constructor(age) {
        this._age = age;
    }

    _incrementAge() {
        this._age += 1;
    }
}
```

但是命名约定在代码中仍然会令人混淆并且并不会真正的保持私有变量不被访问。现在，我们可以使用 **WeakMaps**  来存储变量：
```
let _age = new WeakMap();
class Person {
    constructor(age) {
        _age.set(this, age);
    }

    incrementAge() {
        let age = _age.get(this) + 1;
        _age.set(this, age);
        if (age > 50) {
            console.log('Midlife crisis');
        }
    }
}
```
在 WeakMaps 存储变量很酷的一件事是它的 key 他不需要属性名称，可以使用 ``Reflect.ownKeys()`` 来查看这一点：
```
> const person = new Person(50);
> person.incrementAge(); // 'Midlife crisis'
> Reflect.ownKeys(person); // []
```

一个更实际的实践就是可以 WeakMaps 储存 DOM 元素，而不会污染元素本身：
```
let map = new WeakMap();
let el  = document.getElementById('someElement');

// Store a weak reference to the element with a key
map.set(el, 'reference');

// Access the value of the element
let value = map.get(el); // 'reference'

// Remove the reference
el.parentNode.removeChild(el);
el = null;

// map is empty, since the element is destroyed
```

如上所示，当一个对象被垃圾回收机制销毁的时候， WeakMap 将会自动地一处关于这个对象地键值对。

> **注意**：为了进一步说明这个例子的实用性，可以考虑 jQuery 是如何实现缓存一个对象相关于对引用地 DOM 元素对象。使用 jQuery ，当一个特定地元素一旦在 document 中移除的时候，jQuery 会自动地释放内存。总体来说，jQuery 在任何 dom 库中都是很有用的。

## Promises
Promises 可以让我们远离平行的代码（回调地狱）：
```
func1(function (value1) {
    func2(value1, function (value2) {
        func3(value2, function (value3) {
            func4(value3, function (value4) {
                func5(value4, function (value5) {
                    // Do something with value 5
                });
            });
        });
    });
});
```

转变成垂直代码：
```
func1(value1)
    .then(func2)
    .then(func3)
    .then(func4)
    .then(func5, value5 => {
        // Do something with value 5
    });
```

在 ES6 之前，我们使用 [bluebird](https://github.com/petkaantonov/bluebird) 或者 [q](https://github.com/kriskowal/q)，现在我们可以使用原生的 **Promise** 了。
```
new Promise((resolve, reject) =>
    reject(new Error('Failed to fulfill Promise')))
        .catch(reason => console.log(reason));
```

我们有两个处理器，``resolve``（当Promise是 ``fulfilled`` 时的回调）和 ``reject``（当Promise是 ``rejected`` 时的回调）：。

> **Promises的好处**：对错误的处理使用一些列回调会使代码很混乱，使用 Promise ，我看可以清晰的让错误冒泡并且在合适的时候处理它，甚至，在 Promise 确定了  resolved/rejected 之后，他的值是不可改变的－－它从来不会变化。

这是使用 Promise 的一个实际的栗子：
```
var request = require('request');

return new Promise((resolve, reject) => {
  request.get(url, (error, response, body) => {
    if (body) {
        resolve(JSON.parse(body));
      } else {
        resolve({});
      }
  });
});
```

我们还可以使用 ``Promise.all()`` 来 ``并行`` 处理多个异步函数：
```
let urls = [
  '/api/commits',
  '/api/issues/opened',
  '/api/issues/assigned',
  '/api/issues/completed',
  '/api/issues/comments',
  '/api/pullrequests'
];

let promises = urls.map((url) => {
  return new Promise((resolve, reject) => {
    $.ajax({ url: url })
      .done((data) => {
        resolve(data);
      });
  });
});

Promise.all(promises)
  .then((results) => {
    // Do something with results of all our promises
 });
```

## Generators 生成器
就像 Promises  可以帮我们避免回调地狱，Generators 可以帮助我们让代码风格更整洁－－用同步的代码风格来写异步代码，它本质上是一个可以暂停计算并且可以随后返回表达式的值的函数。

一个简单的栗子使用 generators：
```
function* sillyGenerator() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
}

var generator = sillyGenerator();
> console.log(generator.next()); // { value: 1, done: false }
> console.log(generator.next()); // { value: 2, done: false }
> console.log(generator.next()); // { value: 3, done: false }
> console.log(generator.next()); // { value: 4, done: false }
```
`next` 可以回去到下一个 `yield` 返回的值，当然上面的代码是非常不自然的，我们可以利用 Generators 来用同步的方式来写异步操作：
```
// Hiding asynchronousity with Generators

function request(url) {
    getJSON(url, function(response) {
        generator.next(response);
    });
}
```
这里的 generator 函数将会返回需要的数据：
```
function* getData() {
    var entry1 = yield request('http://some_api/item1');
    var data1  = JSON.parse(entry1);
    var entry2 = yield request('http://some_api/item2');
    var data2  = JSON.parse(entry2);
}
```
通过 `yield`，我们可以保证 `entry1` 有 `data1` 中我们需要解析并储存的数据。

虽然我们可以利用 Generators 来用同步的方式来写异步操作，但是确认错误的传播变得不再清晰，我们可以在 Generators 中加上 Promise：
```
function request(url) {
    return new Promise((resolve, reject) => {
        getJSON(url, resolve);
    });
}
```

然后我们写一个函数逐步调用 `next` 并且利用 request 方法产生一个 Promise：
```
function iterateGenerator(gen) {
    var generator = gen();
    (function iterate(val) {
        var ret = generator.next();
        if(!ret.done) {
            ret.value.then(iterate);
        }
    })();
}
```

在 Generators 中加上 Promise 之后我们可以更清晰的使用 Promise 中的 ``.catch`` 和 ``reject``来捕捉错误，让我们使用新的 Generator，和之前的还是蛮相似的：
```
iterateGenerator(function* getData() {
    var entry1 = yield request('http://some_api/item1');
    var data1  = JSON.parse(entry1);
    var entry2 = yield request('http://some_api/item2');
    var data2  = JSON.parse(entry2);
});
```

## Async Await
当 ES6 真正到来的时候，``async await`` 可以用更少的处理实现  Promise 和  Generators 所实现的异步处理：
```
var request = require('request');

function getJSON(url) {
  return new Promise(function(resolve, reject) {
    request(url, function(error, response, body) {
      resolve(body);
    });
  });
}

async function main() {
  var data = await getJSON();
  console.log(data); // NOT undefined!
}

main();
```

在 js 引擎中，它所实现的和 Generators 其实是一样的，我更推荐在 ``Generators + Promises`` 之上使用 ``async await``，更多的资源和使用 ES7 和 用 babel 转化可以[看这里](http://masnun.com/2015/11/11/using-es7-asyncawait-today-with-babel.html)。


## Getter/Setter 函数
ES6 已经开始实现了 ``getter`` 和 ``setter`` 函数，比如虾面这个栗子：
```
class Employee {

    constructor(name) {
        this._name = name;
    }

    get name() {
      if(this._name) {
        return 'Mr. ' + this._name.toUpperCase();  
      } else {
        return undefined;
      }  
    }

    set name(newName) {
      if (newName == this._name) {
        console.log('I already have this name.');
      } else if (newName) {
        this._name = newName;
      } else {
        return false;
      }
    }
}

var emp = new Employee("James Bond");

// uses the get method in the background
if (emp.name) {
  console.log(emp.name);  // Mr. JAMES BOND
}

// uses the setter in the background
emp.name = "Bond 007";
console.log(emp.name);  // Mr. BOND 007  
```

最新版本的浏览器也在对象中实现了 ``getter`` 和 ``setter`` 函数，我们可以使用它们来实现 **计算属性**，在设置和获取一个属性之前加上监听器和处理。
```
var person = {
  firstName: 'James',
  lastName: 'Bond',
  get fullName() {
      console.log('Getting FullName');
      return this.firstName + ' ' + this.lastName;
  },
  set fullName (name) {
      console.log('Setting FullName');
      var words = name.toString().split(' ');
      this.firstName = words[0] || '';
      this.lastName = words[1] || '';
  }
}

person.fullName; // James Bond
person.fullName = 'Bond 007';
person.fullName; // Bond 007
```
