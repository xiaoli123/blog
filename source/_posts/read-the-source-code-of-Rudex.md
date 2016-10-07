title: 解析 Redux 源码
date: 2016-10-07 22:21:43
tags:
- JavaScript
- Redux
categories: [JavaScript]
bgimage: /img/bg-redux.png
---

![redux](/img/redux.jpeg)

# TIP

Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

作为 React 全家桶的一份子，Redux 可谓说也是名声响响，在 2016 年学习 JavaScript 想必没有多少人没听过吧。

这里，本文不是来教大家如何使用 Redux 的 API 的，这一类的文章已经很多，对于 Redux 的介绍和学习可以点击下列链接：

- [官方文档](http://redux.js.org/)
- [官方Github](https://github.com/reactjs/redux)
- [中文文档](http://cn.redux.js.org/index.html)
- [redux 大法好 —— 入门实例 TodoList(本人的渣渣文章)](http://qiutc.me/post/redux-%E5%A4%A7%E6%B3%95%E5%A5%BD-%E2%80%94%E2%80%94-%E5%85%A5%E9%97%A8%E5%AE%9E%E4%BE%8B-TodoList.html)

Redux 体小精悍（只有2kB）且没有任何依赖，因此本文想通过阅读 Redux 的源码来学习 Redux 的使用以及思想。

# 源码结构

Redux 的源码结构很简单，我们可以直接看 `src` 目录下的代码：

```
.src
├── utils                #工具函数
├── applyMiddleware.js
├── bindActionCreators.js        
├── combineReducers.js     
├── compose.js       
├── createStore.js  
└── index.js             #入口 js
```

# index.js

这个是整个代码的入口：

```javascript
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'

function isCrushed() {}

if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
    '。。。'
  )
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose
}
```

这里的 `isCrushed` 函数主要是为了验证在非生产环境下 Redux 是否被压缩（因为如果被压缩了那么 `(isCrushed.name !== 'isCrushed')` 就是 `true`），如果被压缩会给开发者一个 warn 提示）。

然后就是暴露 `createStore` `combineReducers` `bindActionCreators` `applyMiddleware` `compose` 这几个接口给开发者使用，我们来逐一解析这几个 API。

# createStore.js

这个是 Redux 最主要的一个 API 了，它创建一个 Redux store 来以存放应用中所有的 state，应用中应有且仅有一个 store。

```javascript
import isPlainObject from 'lodash/isPlainObject'
import $$observable from 'symbol-observable'

// 私有 action
export var ActionTypes = {
  INIT: '@@redux/INIT'
}

export default function createStore(reducer, preloadedState, enhancer) {
  // 判断接受的参数个数，来指定 reducer 、 preloadedState 和 enhancer
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  // 如果 enhancer 存在并且适合合法的函数，那么调用 enhancer，并且终止当前函数执行
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  // 储存当前的 currentReducer
  var currentReducer = reducer
  // 储存当前的状态
  var currentState = preloadedState
  // 储存当前的监听函数列表
  var currentListeners = []
  // 储存下一个监听函数列表
  var nextListeners = currentListeners
  var isDispatching = false

  // 这个函数可以根据当前监听函数的列表生成新的下一个监听函数列表引用
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  ... getState ...

  ... subscribe ...

  ... dispatch ...

  ... replaceReducer ...

  ... observable ...

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```

首先定义了一个 `ActionTypes` 对象，它是一个 action，是一个 Redux 的私有 action，不允许外界触发，用来初始化 Store 的状态树和改变 reducers 后初始化 Store 的状态树。

#### createStore

然后着重来看 `createStore` 函数：

##### 接受

它可以接受三个参数，reducer、preloadedState、enhancer：
- reducer：是一个函数，返回下一个状态，接受两个参数：当前状态 和 触发的 action；
- preloadedState：初始状态对象，可以很随意指定，比如服务端渲染的初始状态，但是如果使用 combineReducers 来生成 reducer，那必须保持状态对象的 key 和 combineReducers 中的 key 相对应；
- enhancer：store 的增强器函数，可以指定为 第三方的中间件，时间旅行，持久化 等等，但是这个函数只能用 Redux 提供的 applyMiddleware 函数来生成；

根据传入参数的个数和类型，判断 reducer 、 preloadedState 、 enhancer；

##### 返回

调用完函数它返回的接口是 `dispatch` `subscribe` `getState` `replaceReducer` 和  `[$$observable]`

这也是我们开发中主要使用的几个接口。

##### enhancer

如果 enhancer 参数传入并且是个合法的函数，那么就是调用 `enhancer` 函数（传入 `createStore` 来给它操作），`enhancer` 函数返回的也是一个函数，在这里传入 `reducer` 和 `preloadedState`，并且返回函数调用结果，终止当前函数执行；
在 enhancer 函数里面是如何操作使用的可以看 `applyMiddleware` 部分；

##### getState

```javascript
function getState() {
  return currentState
}
```

这个函数可以获取当前的状态，`createStore` 中的 `currentState` 储存当前的状态树，这是一个闭包，这个参数会持久存在，并且所有的操作状态都是改变这个引用，`getState` 函数返回当前的 `currentState`。

##### subscribe

```javascript
function subscribe(listener) {
  // 判断传入的参数是否为函数
  if (typeof listener !== 'function') {
    throw new Error('Expected listener to be a function.')
  }

  var isSubscribed = true

  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  return function unsubscribe() {
    if (!isSubscribed) {
      return
    }

    isSubscribed = false

    ensureCanMutateNextListeners()
    var index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)
  }
}
```

这个函数可以给 store 的状态添加订阅监听函数，一旦调用 `dispatch` ，所有的监听函数就会执行；
`nextListeners` 就是储存当前监听函数的列表，调用 `subscribe`，传入一个函数作为参数，那么就会给 `nextListeners` 列表 `push` 这个函数；
同时调用 `subscribe` 函数会返回一个 `unsubscribe` 函数，用来解绑当前传入的函数，同时在 `subscribe` 函数定义了一个 `isSubscribed` 标志变量来判断当前的订阅是否已经被解绑，解绑的操作就是从 `nextListeners` 列表中删除当前的监听函数。

##### dispatch

```javascript
function dispatch(action) {
  if (!isPlainObject(action)) {
    throw new Error(
      'Actions must be plain objects. ' +
      'Use custom middleware for async actions.'
    )
  }

  // 判断 action 是否有 type｛必须｝ 属性
  if (typeof action.type === 'undefined') {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
      'Have you misspelled a constant?'
    )
  }

  // 如果正在 dispatch 则抛出错误
  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.')
  }

  // 对抛出 error 的兼容，但是无论如何都会继续执行 isDispatching = false 的操作
  try {
    isDispatching = true
    // 使用 currentReducer 来操作传入 当前状态和action，放回处理后的状态
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false
  }

  var listeners = currentListeners = nextListeners
  for (var i = 0; i < listeners.length; i++) {
    var listener = listeners[i]
    listener()
  }

  return action
}
```

这个函数是用来触发状态改变的，他接受一个 action 对象作为参数，然后 reducer 根据 action 的属性 以及 当前 store 的状态来生成一个新的状态，赋予当前状态，改变 store 的状态；
即 `currentState = currentReducer(currentState, action)`；
这里的 `currentReducer` 是一个函数，他接受两个参数：当前状态 和 action，然后返回计算出来的新的状态；
然后遍历 `nextListeners` 列表，调用每个监听函数；

##### replaceReducer

```javascript
function replaceReducer(nextReducer) {
  // 判断参数是否是函数类型
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer
  dispatch({ type: ActionTypes.INIT })
}
```

这个函数可以替换 store 当前的 reducer 函数，首先直接把 `currentReducer = nextReducer`，直接替换；
然后 `dispatch({ type: ActionTypes.INIT })` ，用来初始化替换后 reducer 生成的初始化状态并且赋予 store 的状态；

##### observable

```javascript
function observable() {
  var outerSubscribe = subscribe
  return {
    subscribe(observer) {
      if (typeof observer !== 'object') {
        throw new TypeError('Expected the observer to be an object.')
      }

      function observeState() {
        if (observer.next) {
          observer.next(getState())
        }
      }

      observeState()
      var unsubscribe = outerSubscribe(observeState)
      return { unsubscribe }
    },

    [$$observable]() {
      return this
    }
  }
}
```

对于这个函数，是不直接暴露给开发者的，它提供了给其他观察者模式／响应式库的交互操作，具体可看 [https://github.com/zenparsing/es-observable](https://github.com/zenparsing/es-observable)

##### last

最后执行 `dispatch({ type: ActionTypes.INIT })`，用来根据 reducer 初始化 store 的状态。



# compose.js

`compose` 可以接受一组函数参数，从右到左来组合多个函数，然后返回一个组合函数；

```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}
```



# applyMiddleware.js

```javascript
export default function applyMiddleware(...middlewares) {
  // 这个返回的函数就是 enhancer，接受 createStore 函数，再返回一个函数，接受的其实只有 reducer 和 preloadedState；
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    // 暴漏 getState 和 dispatch 给 第三方中间价使用
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    // 创造第三方中间件使用 middlewareAPI 后返回的函数组成的数组
    chain = middlewares.map(middleware => middleware(middlewareAPI))

    // 结合这一组函数 和 dispatch 组成的新的 dispatch，然后这个暴漏给用户使用，而原有的 store.dispatch 是不变的，但是不暴漏
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

`applyMiddleware` 函数的作用是组合 多个 中间件等等，然后返回一个函数（`enhancer`）

还记得在 `createStore` 中的一段吗：

```javascript
if (typeof enhancer !== 'undefined') {
  if (typeof enhancer !== 'function') {
    throw new Error('Expected the enhancer to be a function.')
  }

  return enhancer(createStore)(reducer, preloadedState)
}
```

这里 `enhancer === applyMiddleware(...)`；

然后再执行 `enhancer(createStore)` 继续之后的操作；

这里 `enhancer(createStore) 等同于 (reducer, preloadedState, enhancer) => { ... }`

然后再执行 `enhancer(createStore)(reducer, preloadedState)`；

再回到 `applyMiddleware` ，这里调用了 `var store = createStore(reducer, preloadedState, enhancer)` ；
如上所见，这里执行的时候已经没有 `enhancer` 参数了，因此会再次执行 `createStore` 函数的全部部分，然后得到一个返回的实例 `store`；

之后会生成一个新的 `dispatch` ，先保存下来原生的 `dispatch` ： `var dispatch = store.dispatch`；

```javascript
var middlewareAPI = {
  getState: store.getState,
  dispatch: (action) => dispatch(action)
}
```

这一步是把 store 的 `getState` 和 `dispatch` 接口暴露给中间件来操作： `chain = middlewares.map(middleware => middleware(middlewareAPI))`；

最后组合 全部中间件的返回值（函数）`chain` 和 `store.dispatch`，然后返回新的 `dispatch` ： `dispatch = compose(...chain)(store.dispatch)`

这里的 `dispatch` 并不是原有的 store 的，而是经过组合中间件之后新的 `dispatch`；

最后返回暴露给用户的接口：

```javascript
return {
  ...store,
  dispatch
}
```

主要还是 store 原有的接口，但是用新的 dispatch 替换了原有的；

**这个函数其实就是根据中间件和store的接口生成新的 dispatch 然后暴露给用户**



# combineReducers.js

这个函数可以组合一组 reducers(对象) ，然后返回一个新的 reducer 函数给 `createStore` 使用。

它接受一组 reducers 组成的对象，对象的 key 是对应 value（reducer函数）的状态名称；

比如： `{ userInfo: getUserInfo, list: getList }`

`userInfo` 是根据 `getUserInfo` 函数计算出来的；

那么 store 里面的 state 结构就会是: `{ userInfo: ..., list: ... }`


```javascript
export default function combineReducers(reducers) {

  // 根据 reducers 生成最终合法的 finalReducers：value 为 函数
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }

  var finalReducerKeys = Object.keys(finalReducers)

  if (process.env.NODE_ENV !== 'production') {
    var unexpectedKeyCache = {}
  }

  // 验证 reducer 是否合法
  var sanityError
  try {
    assertReducerSanity(finalReducers)
  } catch (e) {
    sanityError = e
  }

  // 返回最终生成的 reducer
  return function combination(state = {}, action) {
    if (sanityError) {
      throw sanityError
    }

    if (process.env.NODE_ENV !== 'production') {
      var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache)
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    // 遍历一遍看是否改变，然后返回原有状态值或者新的状态值
    return hasChanged ? nextState : state
  }
}
```

最终返回一个 `combination` 也就是真正传入 `createStore` 的 reducer 函数；

这是一个标准的 reducer 函数，接受一个初始化状态 和 一个 action 参数；

每次调用的时候会去遍历 `finalReducer` （有效的 reducer 列表），获取列表中每个 reducer 对应的先前状态： `var previousStateForKey = state[key]`；
看到这里就应该明白传入的 `reducers` 组合为什么 `key` 要和 store 里面的 state 的 `key` 相对应；

然后得到当前遍历项的下一个状态： `var nextStateForKey = reducer(previousStateForKey, action)` ；
然后把它添加到整体的下一个状态： `nextState[key] = nextStateForKey`

每次遍历会判断整体状态是否改变： `hasChanged = hasChanged || nextStateForKey !== previousStateForKey`

在最后，如果没有改变就返回原有状态，如果改变了就返回新生成的状态对象： `return hasChanged ? nextState : state`。



# bindActionCreators.js

`bindActionCreators` 函数可以生成直接触发 action 的函数；

实质上它只说做了这么一个操作 `bindActionFoo = (...args) => dispatch(actionCreator(...args))`

因此我们直接调用 `bindActionFoo` 函数就可以改变状态了；

接受两个参数，一个是 `actionCreators（` actionCreator 组成的对象，key 对于生成的函数名／或者是一个 actionCreator ），一个是 `dispatch，` store 实例中获取；

```javascript
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

export default function bindActionCreators(actionCreators, dispatch) {
  // 是一个函数，直接返回一个 bindActionCreator 函数，这个函数调用 dispatch 触发 action
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
      `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  // 遍历对象，然后对每个遍历项的 actionCreator 生成函数，将函数按照原来的 key 值放到一个对象中，最后返回这个对象
  var keys = Object.keys(actionCreators)
  var boundActionCreators = {}
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i]
    var actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```

# 总结
阅读了一遍 Redux 的源码，实在是太精妙了，少依赖，少耦合，纯函数式。
