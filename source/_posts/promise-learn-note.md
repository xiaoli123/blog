title: Promise 对象初尝试
date: 2016-03-27 15:06:49
tags:
- Note
- JavaScript
categories: [JavaScript]
---

![Promise](/img/promise-1.png)
# 浏览器支持
![Promise](/img/promise-3.jpg)
http://caniuse.com/promises/embed/agents=desktop

# What is Promise?
Promise是抽象异步处理对象以及对其进行各种操作的组件。

说到 javascript 异步操作，可能想到的是这样：
```
// 以 jQuery 的 ajax 为例
$.get('/get_url', function(result, status) {
	if(status == 'success') {
		alert('success');
	}
	if(status == 'error') {
		alert('error');
	}
});
```
对于 ajax 的 get 操作来说，是一个异步的过程，通过回调函数，在得到返回的时候才会去执行操作。

但是试想一下当操作越来越多，回调里面还要回调的时候，一层层回调函数是不是让人抓狂，不论在代码可读性还是编写效率来看都是很麻烦的。
看一下我们用 Promise 可以怎么做一个异步的操作：
```
// 这个 getData 是我们预先实例好的一个 Promise 对象，如何处理这个对象我们这里不讨论
var promise = getData('/get_url');
promise.then(function(result) {
	console.log(result);
}).catch(function(error) {
  console.log(error);
});
```
这样的风格是不是会更好呢，执行一个 promise，然后 then 里面传入回调函数，如果愿意，我们可以在 then 后面再更很多个 then，catch 可以捕捉错误，看起来代码清晰简明多了。
所以，promise的功能是可以将复杂的异步处理轻松地进行模式化。

#构造函数 Constructor
```
new Promise(executor);
new Promise(function(resolve, reject) { ... });
```
这里的 ``executor`` 是我们实例化一个 promise 对象时应该传入的参数，这个参数只一个函数，这个函数接受两个参数 ``resolve``和``reject``。
两个方法：
- ``resolve(result)`` 在 promise 中执行这个方法表示成功，会在执行之后执行后面的 then 所传入的函数，它接受到的参数也会被 then 里面的函数接受到，一般来说参数为执行结果成功时候的数据；
- ``reject(error)`` 在 promise 中执行这个方法表示失败，他一般接受一个 error 错误参数，会被后面的 catch 捕捉到错误执行。
demo：
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve('success');
    }, 2000);
  });
};
testFoo().then(function(result) {
  console.log(result);
}).catch(function(error) {
  console.log(error);
});
```
在这里我们定义了一个 ``testFoo`` 函数，这个函数返回一个``Promise``的实例化对象，两秒之后会执行``resolve('success');``，表示成功，传入一个参数，在两秒之后，我们then里面传入的函数执行了，接收到了刚刚那个参数；但是catch里面的函数并没有执行，因为我们没有在 promise 里面执行拒绝操作。

如果我们在四秒之后执行 ``reject`` 操作呢：
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve('success');
    }, 2000);
    setTimeout(function() {
      reject('error');
    }, 4000);
  });
};
testFoo().then(function(result) {
  console.log(result);
}).catch(function(error) {
  console.log(error);
});
```
貌似只出现``resolve``的结果，因为一个 promise 没办法做多次结果操作。
我们就这样：
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      reject('error');
    }, 4000);
  });
};
testFoo().then(function(result) {
  console.log(result);
}).catch(function(error) {
  console.log(error);
});
```
现在结果如我们所预料了。


# PromiseStatus 状态
状态分为三种：
- ``fulfilled`` - Fulfilled 已完成，在 ``resolve``时，调用 then 的 ``onFulfilled``函数；
- ``Rejected`` - Rejected 拒绝，在``reject``时，调用 then 的 ``onRejected``函数，或者 catch 里面的函数；
- ``unresolved`` - Pending 等待，是 promise 初始化的时候的状态。

promise 的 初始化状态为 unresolved，根据异步结果变为 fulfilled 或者 Rejected，一旦变为其中一个就不可改变，这也是我们之前上面为什么执行了 resolve 之后再执行 reject 而没有结果的原因了。

# 方法概述
## 快捷方法
- ``Promise.resolve()`` 这个是promise的静态方法
```
Promise.resolve(10).then(function(value){
    console.log(value);
});
```
这个方法会让 Promise 立即进入 fulfilled 状态，一般用来测试用。
- ``Promise.reject()``相应着我们有这个方法
```
Promise.reject('err').catch(function(err){
    console.log(err);
});
```
## 实例方法
- ``then(onFulfilled, onRejected)``这个方法具体是这样的，传入两个函数，一个处理fulfilled状态，另一个处理Rejected状态，一般使用我们只传入第一个函数，第二个放在 catch 来处理。
- ``catch(onRejected)``处理Rejected状态，可以这么理解``catch(onRejected)``=``promise.then(undefined, onRejected)``。

# 链式调用
看看这个 demo:
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(1);
    }, 2000);
  });
};
testFoo().then(function(result) {
  console.log(result);
  return ++result;
}).then(function(result) {
  console.log(result);
  return ++result;
}).then(function(result) {
  console.log(result);
  return ++result;
}).catch(function(error) {
  console.log(error);
});

// 1
// 2
// 3
```
可以看见结果，这个方法的流程是什么样的呢？
![task](/img/promise-2.png)
第一个 then 函数和之前讲的一样，处理 promise 的 fulfilled，第二个 then 的函数是处理前一个 then 函数处理完的结果，他们之间参数传递的途径是前一个 then 函数 return 一个数据，然后后一个 then 函数接受到这个参数。
如果你愿意，可以再后面加很多很多个 then，他们的流程主要是这样。

如果我们把 catch 提前呢？
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(1);
    }, 2000);
  });
};
testFoo().then(function(result) {
  console.log(result);
  return ++result;
}).then(function(result) {
  console.log(result);
  return ++result;
}).catch(function(error) {
  console.log(error);
}).then(function(result) {
  console.log(result);
  return ++result;
});

// 1
// 2
// 3
```
可以看出，结果一样，说明catch只会在发生 reject 的时候调用。
那如果在中间的一个 then 中抛出一个异常呢？
```
var testFoo = function() {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(1);
    }, 2000);
  });
};
testFoo().then(function(result) {
  console.log(result);
  return ++result;
}).then(function(result) {
  console.log(result);
  throw new Error("throw Error")
  return ++result;
}).then(function(result) {
  console.log(result);
  return ++result;
}).catch(function(error) {
  console.log(error);
});
// 1
// 2
// Error: throw Error
```
我们在第二个then中抛出一个异常，而后立即被 catch 捕捉到，第三个 then 并没有执行。

到这里我们想一下从then的传参和捕捉异常来看，新加一个 then 只是注册了一个回调函数那么简单吗？
不不不，每次调用then都会返回一个新创建的promise对象，这就解释了上面的一切原因。

# 并发调用
试想一个场景，我们执行多个异步操作（ajax等等），但是我们想在这几个操作都完成的时候才去执行一个函数。
如果按照传统的方法来做，代码会很乱很散，关键不优雅。让我们尝试用 promise 。
先看这个 demo
```
var testFoo = function(time, value) {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(value);
    }, time * 1000);
  });
};
var tasks = {
  task1: function() {
    return testFoo(1, 2);
  },
  task2: function() {
    return testFoo(1.3, 3);
  },
  task3: function() {
    return testFoo(1.5, 1);
  }
};
var main = function() {
  function recordValue(results, value) {
    results.push(value);
    console.log(value);
    console.log(results);
    return results;
  }
  var pushValue = recordValue.bind(null, []);
  return tasks.task1().then(pushValue).then(tasks.task2).then(pushValue).then(tasks.task3).then(pushValue);
};
main().then(function(value) {
  console.log(value);
});

// [2, 3, 1]
```
这么实现明显看起来凌乱，特别对于 main 函数，可读性也比较差。
那么有没有更优雅的方法呢，答案：有！😄。

## Promise.all
``Promise.all`` 方法接受一个以 promise 对象为元素的数组，在全部执行操作完成后才回调用then里面的方法，看代码:
```
var testFoo = function(time, value) {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(value);
    }, time * 1000);
  });
};
var tasks = {
  task1: function() {
    return testFoo(1, 2);
  },
  task2: function() {
    return testFoo(1.3, 3);
  },
  task3: function() {
    return testFoo(1.5, 1);
  }
};
var main = function() {
  return Promise.all([tasks.task1(), tasks.task2(), tasks.task3()]);
}
main().then(function(result) {
  console.log(result);
});

// [2, 3, 1]
```
我们吧要执行的 promise 对象作为数组的元素传给 ``Promise.all()``，``Promise.all().then()``里面定义的函数接受到一个数组，元素是这几个操作返回的值，顺序和 promise 对象放入的顺序一样，比如第一个 promise 对象返回的值是2，那么结果的第一个元素就是2。
并且每一个promise是同时执行的－－并发。
在所有 promise 的状态为 FulFilled 的时候才会去执行 then 里面的函数。
那么捕捉异常呢？
修改一下例子：
```
var testFoo = function(time, value, err) {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      if(err) {
	    reject(err);
		return false;
	  }
      resolve(value);
    }, time * 1000);
  });
};
var tasks = {
  task1: function() {
    return testFoo(1, 2, 'error');
  },
  task2: function() {
    return testFoo(1.3, 3, 'error1');
  },
  task3: function() {
    return testFoo(1.5, 1);
  }
};
var main = function() {
  return Promise.all([tasks.task1(), tasks.task2(), tasks.task3()]);
}
main().then(function(result) {
  console.log(result);
}).catch(function(err) {
  console.log(err);
});

// [2, 3, 1]
```
我们让其中2 promise 个抛出异常，看到捕捉到了那个异常，而且是捕捉到第一个异常就停止了。
说明 all 的操作是当所有 promise 状态为 FulFilled 的时候才会执行 then 的操作。而一旦有一个 Rejected 就catch这个异常，并且停止。

## Promise.race
他和 Promise.all 一样，接受一个 promise 对象组成的数组，也是并发执行，但是 Promise.race 是只要有一个promise对象进入 FulFilled 或者 Rejected 状态的话，就会继续进行后面的处理。
```
var testFoo = function(time, value) {
	return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(value);
    }, time * 1000);
  });
};
var tasks = {
  task1: function() {
    return testFoo(1, 2);
  },
  task2: function() {
    return testFoo(1.3, 3);
  },
  task3: function() {
    return testFoo(1.5, 1);
  }
};
var main = function() {
  return Promise.race([tasks.task1(), tasks.task2(), tasks.task3()]);
}
main().then(function(result) {
  console.log(result);
});

// 2
```
可以看到，task1 最先完成，然后就拿到他的值进行 then 操作。
