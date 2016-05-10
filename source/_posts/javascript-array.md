title: javascript 中的数组使用
date: 2016-04-29 20:49:51
tags:
- JavaScript
categories: [JavaScript]
bgimage: http://7xlkda.com1.z0.glb.clouddn.com/type-away-numero-dos.jpg
---
# tip
系统学习 Array 对象的笔记。

![js](/img/js.jpeg)

# 定义
数组的标准定义：一个存储元素的线性集合，元素可以通过索引来任意存取，索引通常是数字，用来计算元素之间存储位置的偏移量。
在javascript中的数组有些不同，他们其实是特殊的对象，表示偏移量的索引是数组对象的属性（字符串）。效率上不如其他语言高。

# 使用数组

## 创建数组
- 通过  ``var arr1 = [1,2,3]``
- 使用 new ``var arr2 = new Array(1,2,3)``（注意：如果里面传入的是一个数字，比如 10 ，是定义一个 length 为10的数组，每一个元素都是 ``undefined``）

在这里创造的数组实际上都是一个对象，然后把对象的引用赋值给变量，实质上我们对数组的操作都是对引用进行操作。
## 读写数组
```
// 读取数组
var arr1 = [1,2,3,4];
for(var i = 0; i < arr1.length; i++) {
	console.log(arr1[i]);
}
// 改变数组值
var arr2 = [];
for(var i = 0; i < 10; i++) {
	arr2[i] = i;
}
```
我们可以用``[]``读取到索引的数组，同时也可以对他进行赋值改变的操作。

## 由字符串生成数组
可以调用字符串对象的 ``split()`` 方法通过一个字符串生成一个数组。
```
var sentence = 'hello world haha hahahha';
var words = sentence.split(' ');
console.log(words);
// ["hello", "world", "haha", "hahahha"]
```
我们用空格号来切分了``sentence``这个字符串。

## 对数组进行整体性操作
把一个数组赋给另一个数组。
```
var arr1 = [1,2,3,4];
var arr2 = arr1;
```
这里 arr2 等于 arr1 了，但是别忘了，在 javascript 里面数组是一个对象，所以赋值的只是一个新的引用！如果我们修改了 arr2 会怎么样呢？
```
var arr1 = [1,2,3,4];
var arr2 = arr1;
arr2[4] = 5;
console.log(arr1);
// [1, 2, 3, 4, 5]
```
看见没有，arr2 改变了 arr1 也改变了，他们都是指向同一个数组对象，这种复制叫做浅复制。
现在我们有一个更好的方案－深复制：
```
function copyArr(arr) {
	var _arr = [];
	for(var i = 0; i < arr.length; i++) {
		_arr[i] = arr[i];
	}
	return _arr;
}
var arr0 = [1,2,3,4];
var arr1 = copyArr(arr0);
console.log(arr1);
// [1, 2, 3, 4]
arr1[4] = 5;
console.log(arr0);
// [1, 2, 3, 4]
console.log(arr1);
// [1, 2, 3, 4, 5]
```
可以看到我们用循环遍历把 arr0 的元素复制给 arr1 的对应索引量的元素，改变 arr1，arr0 不改变。
当然这还是存在一些问题，比如这是个二维数组呢？数组嵌套再嵌套数组呢？可以在for循环里面判断元素是否数组，然后递归。

# 存取函数

## 查找元素
``indexOf()``是最常用的了，他接受一个参数，然后查找数组中元素等于这个参数值的第一个元素，返回对应的索引值，如果没有找到这个参数，返回``-1``，如果由多个，返回第一个的索引值。
```
var arr = [1,2,3,4,3,5];
console.log( arr.indexOf(2) );
// 1
console.log( arr.indexOf(6) );
// -1
console.log( arr.indexOf(3) );
// 2
```
还有一个对应的函数是``lastIndexOf()``他是从后面查找起的，对于多个，会返回查找到的第一个的索引值。
```
var arr = [1,2,3,4,3,5];
console.log( arr.lastIndexOf(2) );
// 1
console.log( arr.lastIndexOf(6) );
// -1
console.log( arr.lastIndexOf(3) );
// 4
```

## 数组的字符串表示
把数组转换成一个字符串有两个方法：``toString()`` 和``join()``前者是生成用``,``分割开每个元素的字符串，后者可以传入一个参数来指导用这个参数值来隔开，上代码：
```
var arr = ['a', 'b', 'c'];
console.log( arr.toString() ); // a,b,c
console.log( arr.join('-') ); // a-b-c
```

## 由已有的数组创建新数组
方法有 ``concat()``和``splice()``和``slice()``
- ``concat()`` 接受多个数组作为参数，返回和他们合并的一个数组；
- ``splice()``截取一个数组的子集返回一个新数组；（会改变原数组）
- ``slice()``截取一个数组的子集：
```
var arr1 = [1,2,3,4];
var arr2 = [5,6];
var arr3 = [8];
var arr4 = arr1.concat(arr2, arr3);
console.log(arr4);
// [1, 2, 3, 4, 5, 6, 8]
```
```
var arr1 = [1,2,3,4,5,6,7,8];
var arr2 = arr1.splice(2, 4); // 第一个参数是截取位置开始的索引值，第二个是截取的长度
console.log(arr2);
// [3, 4, 5, 6]

var arr3 = [1,2,3,4];
var arr4 = arr3.slice(1,3);
// [2, 3]
```
# 可变函数
## 为数组增减元素
### 添加
- ``push()``将一个元素添加到数组的末尾，返回被添加的元素；
- ``unshift()``将一个元素添加到数组开头，返回被添加的元素；

```
var arr = [1,2,3,4];
arr.push(5); // 5
console.log(arr);
// [1, 2, 3, 4, 5]
arr.unshift(0); // 0
console.log(arr);
// [0, 1, 2, 3, 4, 5]
```
### 删除
- ``pop()``删除数组末尾的一个元素，返回这个元素值；
- ``shift()``删除数组开头的一个元素，返回这个元素值；

```
var arr = [1,2,3,4];
arr.pop(); // 4
console.log(arr);
// [1, 2, 3]
arr.shift(); // 1
console.log(arr);
// [2, 3]
```

### 在数组中间添加或者删除元素
还是 ``splice``方法，他接受参数：
+ 启始索引
+ 需要删除的数组个数（添加元素的时候可以是0）
+ 需要添加进数组的元素

```
// 添加
var arr1 = [1,2,3,4,5];
arr1.splice(2, 0, 10);
console.log(arr1);
// [1, 2, 10, 3, 4, 5]

// 删除
var arr2 = [1,2,3,4,5];
arr2.splice(2, 1);
console.log(arr2);
// [1, 2, 4, 5]
```

## 为数组排序
- ``reverse()``可以使数组反转；
- ``sort()``排序，可以接受一个排序规则函数作为参数；

```
var arr = [1,2,3,4];
arr.reverse();
console.log(arr);
// [4, 3, 2, 1]
```

```
var arr = [23, 11, 8, 24, 55, 43];
arr.sort(function(a1, a2) {
	return a1 - a2;
});
// [8, 11, 23, 24, 43, 55]
```

# 迭代方法
## 不生成新的数组的迭代器方法
- ``forEach()``，他接受一个函数作为参数，函数接受两个参数：当前元素值、当前元素索引，单纯遍历数组：
```
var arr = [10,20,30,40,50,60];
arr.forEach(function(item, idx) {
	console.log(item);
	console.log(idx);
});
```
- ``every()``接受一个返回值为布尔类型的函数为参数，对数组中每个元素使用该函数，如果全都是返回``true``则该方法返回``true``，如果有一个以上返回``false``，那么该方法返回``false``；
```
var arr = [1,2,3,4,5,6];
var b1 = arr.every(function(item, idx) {
	return item >= 1;
});
console.log(b1); // true
var b2 = arr.every(function(item, idx) {
	return item >= 2;
});
console.log(b2); // false
```
- ``some()``方法，其实和``every()``方法很像，顾名思义，如果返回的有一个结果是``true``则返回``true``，如果全是``false``，则返回``false``;
```
var arr = [1,2,3,4,5,7];
var b1 = arr.some(function(item, idx) {
	return item > 5;
});
console.log(b1); // true
var b2 = arr.some(function(item, idx) {
	return item > 8;
});
console.log(b2); // false
```
- ``reduce()``方法，他接受两个参数：一个函数、一个初始值（可不传），函数接受4个参数：之前一次遍历的返回值（第一次的话就是初始值）、当前元素值、当前元素索引、被遍历的数组，函数返回一个值。他会从一个累加之开始不断对后面的元素调用该函数；
```
var arr = [1,2,3,4,5,6];
var sum = arr.reduce(function(runningTotal, currentValue) {
	return runningTotal + currentValue;
}, 10);
console.log(sum); // 31
```
- ``reduceRight()``方法，它其实和``reduce()``方法一样，区别是他是从数组的右边开始累加到左边的。

## 生成新数组的方法
- ``map()``方法接受一个函数，这个函数处理每一个元素并返回一个新的值，然后方法会返回这些值组成的一个新数组。
```
var arr = [1,2,3,4,5];
var arr2 = arr.map(function(item, idx) {
	return item + idx;
});
console.log(arr2);
// [1, 3, 5, 7, 9]
```

- ``filter()``方法接受一个函数，函数返回一个布尔类型的值，最后方法返回的数组是被函数返回``true``的元素的集合，顾名思义，就是过滤。
```
var arr = [1,2,3,4,5,6,7,8];
var arr2 = arr.filter(function(item, idx) {
	return item % 2 === 0;
});
console.log(arr2);
// [2, 4, 6, 8]
```


# 二维数组和多维数组

## 创建二维数组
二维数组类似于一种行和列组成的数据表格。
在javascript里面定义二维数组，可以扩展javascript的数组对象：
```
Array.matrix = function(numrows, numcols, initial) {
	var arr = [];
	for(var i = 0; i < numrows; i++) {
		var colnums = [];
		for(var j = 0; j < numcols; j++) {
			colnums[j] = initial;
		}
		arr[i] = colnums;
	}
	return arr;
}
var nums = Array.matrix(3,4,9);
```
## 处理二位数组中的元素
```
var arrs = [[1,2,3], [33,44,55], [12,23,34], [999, 000, 666]];
var total = 0;
for(var row = 0; row < arrs.length; row++) {
	for(var col = 0; col < arrs[row].length; col++) {
		total += arrs[row][col];
	}
}
console.log(total); // 1872
```

# 是否改变原数组
+ 1.不改变原有数组：
  - indexOf
  - lastIndexOf
  - toString
  - join
  - concat
  - slice
  - some
  - forEach
  - every
  - filter
  - map
  - reduce
  - reduceRight
<br>
+ 2.改变数组：
  - splice
  - push
  - pop
  - shift
  - unshift
  - reverse
  - sort

# es6 中的新特性
### Array.from()
``Array.from``方法可以将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）。
下面是一个类似数组的对象，Array.from将它转为真正的数组。
```
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```
### Array.of()
``Array.of``方法用于将一组值，转换为数组。
这个方法的主要目的，是弥补数组构造函数Array()的不足。因为参数个数的不同，会导致Array()的行为有差异。
就是之前说的在用 new 构造一个数组的时候，如果传入的是一个数字，那么返回的是一个该长度的数组：
```
var arr0 = new Array(2);
console.log(arr0);
// []
arr0.length; // 2
```
如果使用 ``Array.of``:
```
var arr0 = Array.of(2);
console.log(arr0);
// [2]

var arr1 = Array.of(1,2,3);
console.log(arr1);
// [1,2,3]
```

### 数组实例的copyWithin()
数组实例的copyWithin方法，在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。也就是说，使用这个方法，会修改当前数组。

它接受三个参数：
- target（必需）：从该位置开始替换数据。
- start（可选）：从该位置开始读取数据，默认为0。如果为负值，表示倒数。
- end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数。

```
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
```

### 数组实例的find()和findIndex()
数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。
```
[1, 4, -5, 10].find((n) => n < 0)
// -5
```
数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。
```
[1, 4, -5, 10].find((n) => n < 0)
// 2
```

### 数组实例的fill()
``fill``方法使用给定值，填充一个数组。
```
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```
fill方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。
```
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
```

### 数组实例的entries()，keys()和values()
ES6提供三个新的方法——entries()，keys()和values()——用于遍历数组。
```
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

### 数组实例的includes()
``Array.prototype.includes``方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。该方法属于ES7，但Babel转码器已经支持。
```
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, NaN].includes(NaN); // true
```
该方法的第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。
```
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```
