title: redux 大法好 —— 入门实例 TodoList
date: 2015-12-21 14:24:43
tags:
- FE
- redux
- react
categories: [redux]
---
![redux](/img/redux.jpeg)
# Tip
前端技术真是日新月异，搞完 React 不搭配个数据流都不好意思了。
满怀期待的心去翻了翻 flux，简直被官方那意识流的文档折服了，真是又臭又长还是我智商问题？😖
转战 redux ，越看越有意思，跟着文档做了个 TodoList 的入门小例子。

废话不多说，先贴上文章用到例子的源码 [https://github.com/TongchengQiu/TodoList-as-redux-demo](https://github.com/TongchengQiu/TodoList-as-redux-demo)
redux 的 Github 仓库 [https://github.com/rackt/redux](https://github.com/rackt/redux)
还有个中文的 gitbook 翻译文档 [http://camsong.github.io/redux-in-chinese/index.html](http://camsong.github.io/redux-in-chinese/index.html)

# advantage
随着spa（不是SPA，是单页应用）的发展，以 react 来说，组件化和状态机的思想真是解放了烦恼的 dom 操作，一切都为状态。state 来操纵 views 的变化。
然而，因为页面的组件化，导致每个组件都必须维护自身的一套状态，对于小型应用还好。
但是对于比较大的应用来说，过多的状态显得错综复杂，到最后难以维护，很难清晰地组织所有的状态，在多人开发中也是如此，导致经常会出现一些不明所以的变化，越到后面调试上也是越麻烦，很多时候 state 的变化已经不受控制。对于组件间通行、服务端渲染、路由跳转、更新调试，我们很需要一套机制来清晰的组织整个应用的状态，redux 应然而生，这种数据流的思想真是了不起。

# state 根对象的结构
在 react 中，我们尽量会把状态放在顶层的组件，在顶层组件使用 redux 或者 router。
这就把组件分为了两种：容器组件和展示组件。
容器组件：和 redux 和 router 交互，维护一套状态和触发 action。
展示组件：展示组件是在容器组件的内部，他们不维护状态，所有数据通过 props 传给他们，所有操作也是通过回调完成。
这样，我们整套应用的架构就显得清晰了。

# part
redux 分为三大部分，store ， action ，reducer 。

## store
整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。
或者这么说 store 的指责有这些：
+ 维护整个应用的 state
+ 提供 getState() 方法获取 state；
+ 提供 dispatch(action) 方法更新 state；
+ 通过 subscribe(listener) 注册监听器。

这么解释一下，整个应用的 state 都储存在 store 中，容器组件可以从 store 中获取所需要的状态。
容器组件同时也可以发送给 store 一个 action，告诉他改变某个状态的值，所以说容器组件只要发送一个指令，就可以叫 store 去 setState，然后 store 的 state 改变，回过来容器组件获取到的 state 改变，导致 views 的更新。

## action
action 可以理解为一种指令，store 数据的唯一由来就是 action，action 是一个对象，它需要至少一个元素，type，type 是这个指令的唯一标识，其它元素是传送这个指令的 state 值
```
{
  type: ACTION_TYPE,
  text: “content”,
}
```
这个指令由组件触发，然后传到 reducer。

## reducer
action 只是说明了要去做什么，和做这件事情需要的参数值。
具体去改变 store 中的 state 是由 reducer 来做的。
reducer 其实是一个包含 switch 的函数，前面不是说组件触发的 action 会传递到 reducer，reducer 接收这个参数 action，他通过 switch(action.type) 然后做不同操作，前面说了，这个 type 是指令的标识，reducer 根据这个标识来作出不同的操作。
这个操作是什么呢？
reducer 还接收另一个参数 state，这个是旧的 state。从 action 里面还可以获取到做这个操作需要的 参数值。
这个操作其实就是对原有的 state 和 从 action 中的到的值，来进行操作（结合，删除，...）然后返回一个 新的 state 到 store。

## 数据流
把前面的语言组织一下，整个操作的数据流其实是这样的：

1. store 把整个应用的 state，getState()，dispatch()，subscribe() 传给顶层容器组件；
2. 容器组件和三个部分交互：
  - 内部的展示组件：容器把状态分发给各个组件，把 dispatch（操作数据的函数）以回调的形式分发给各个组件；
  - action：容器获取 action；
  - reducer：容器可以调用 dispatch(action)，这个上面说了，会以回调的形式给下面的子组件，这样就可以根据不同的用户操作，调用不同的 dispatch(action)，执行了这个函数之后，就把 action 传给 reducer，然后看 reducer；
3. reducer 得到容器组件传来的 action 之后，根据 action.type 这个参数执行不同操作，他还会接收到 store 里面的原 state，然后把原 state 和 action 对象里面的其它参数进行操作，然后 return 一个新的对象。
4. reducer return 一个新的对象到 store，store 根据这个新对象，更新应用状态。

－－－－一个循环 ♻️

# connect
Redux 和 React 之间没有关系，他们并补互相依赖，但是 Redux 和 React 搭配起来简直完美。
我们可以通过 react-redux 这个库把他们绑定起来
```
npm install --save react-redux
```
react-redux 提供两个东西 Provider 和 connect。
## Provider
这个 Provider 其实就是一个中间件，他是在原有 App Container 上面再包一层，他的作用就是接收 store 里面的 store 作为 props，将store放在context里，给下面的connect用的。
## connect
这个组件才是真正连接 Redux 和 React，他包在我们的容器组件的外一层，他接收上面 Provider 提供的 store 里面的 state 和 dispatch，传给一个构造函数，返回一个对象，以属性形式床给我们的容器组件。

# 实战 TodoList
这个项目使用 webpack 来构建，想要了解 webpack 的配置可以看我的其它两篇文章：
[如何使用webpack—webpack-howto](http://qiutc.me/post/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8webpack%E2%80%94webpack-howto.html);
[webpack-best-practice-最佳实践-部署生产](http://qiutc.me/post/webpack-best-practice-%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5-%E9%83%A8%E7%BD%B2%E7%94%9F%E4%BA%A7.html).
## 目录结构
```
.
├── app                 #开发目录
|   |   
|   ├──actions          #action的文件
|   |   
|   ├──components       #内部组件
|   |   
|   ├──containers       #容器组件
|   |   
|   ├──reducers         #reducer文件
|   |   
|   ├──stores           #store配置文件
|   |
|   └──index.js         #入口文件
|      
├── dist                #发布目录
├── node_modules        #包文件夹
├── .gitignore     
├── .jshintrc      
├── server.js           #本地静态服务器      
├── webpack.config.js   #webpack配置文件
└── package.json
```
这里，我们只关注我们的 app 开发目录。
## index.js 入口文件
```
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';
import App from './containers/App';
import configureStore from './stores/configureStore';
const store = configureStore();
render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```
这里我们从 react-redux 中获取了一个 Provider 组件，我们把它渲染到应用的最外层。
他需要一个属性 store ，他把这个 store 放在context里，给App(connect)用。
## store
``app/stores.configureStore.js``
```
import { createStore } from 'redux';
import rootReducer from '../reducers';
export default function configureStore(initialState) {
  const store = createStore(rootReducer, initialState);
  if (module.hot) {
    module.hot.accept('../reducers', () => {
      const nextReducer = require('../reducers');
      store.replaceReducer(nextReducer);
    });
  }
  return store;
}
```
他从 redux 拿到 createStore 这个函数，再获取到 rootReducer ；
createStore 函数接收两个参数，(reducer, [initialState])，reducer 毋庸置疑，他需要从 store 获取 state，以及连接到 reducer 交互。
initialState 是可以自定义的一个初始化 state，可选参数。
``module.hot``这个可以不用管，这是 webpack 热加载的处理，你也可以不要他。
## 容器组件
``containers/App.jsx``
```
import React, { Component, PropTypes } from 'react';
import { connect } from 'react-redux';
import {
  addTodo,
  completeTodo,
  setVisibilityFilter,
  VisibilityFilters
} from '../actions';

import AddTodo from '../components/AddTodo';
import TodoList from '../components/TodoList';
import Footer from '../components/Footer';

class App extends Component {
  render() {
    const { dispatch, visibleTodos, visibilityFilter } = this.props;
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          }
        />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index => dispatch(completeTodo(index))}
        />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter => dispatch(setVisibilityFilter(nextFilter))}
        />
      </div>
    );
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  })),
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};


function selectTodos(todos, filter) {
  switch (filter) {
    case VisibilityFilters.SHOW_ALL:
      return todos;
    case VisibilityFilters.SHOW_COMPLETED:
      return todos.filter(todo => todo.completed);
    case VisibilityFilters.SHOW_ACTIVE:
      return todos.filter(todo => !todo.completed);
  }
}

// 这里的 state 是 Connect 的组件的
function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  };
}

export default connect(select)(App);
```
他从 react-redux 获取 connect 连接组件，通过 ``connect(select)(App)`` 连接 store 和 App 容器组件。
select 是一个函数，他能接收到一个 state 参数，这个就是 store 里面的 state，然后通过这个函数的处理，返回一个对象，把对象里面的参数以属性传送给 App，以及附带一个 dispatch。
所以在 App 里面可以：
```
const { dispatch, visibleTodos, visibilityFilter } = this.props;
```
所以 App 通过 connect 的到 state 和 dispatch，把 state 传递给子组件。
dispatch 这个函数可以接收一个 action 参数，然后就会执行 reducer 里面的操作。
比如：
```
text =>
  dispatch(addTodo(text))
```
``addTodo(text)``，这个函数是在 action 里面的到的，可以看 action 的代码，他其实返回一个 action 对象，所以其实就是``dispatch(action)`` 。

## action
``app/actions/index.js``
```
export const ADD_TODO = 'ADD_TODO';
export const COMPLETE_TODO = 'COMPLETE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';
export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE',
};

export function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  };
}

export function completeTodo(index) {
  return {
    type: COMPLETE_TODO,
    index
  };
}

export function setVisibilityFilter(filter) {
  return {
    type: SET_VISIBILITY_FILTER,
    filter
  };
}
```
在声明每一个返回 action 函数的时候，我们需要在头部声明这个 action 的 type，以便好组织管理。
每个函数都会返回一个 action 对象，所以在 容器组件里面 调用
```
text =>
  dispatch(addTodo(text))
```
就是调用``dispatch(action)`` 。

## reducer
``app/reducers/visibilityFilter.js``
```
import {
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from '../actions';

const { SHOW_ALL } = VisibilityFilters;

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter;
    default:
      return state;
  }
}

export default visibilityFilter;
```
这里我们从 actions 获得各个 type 的参数，以便和 action 做好映射对应。
整个函数其实就是执行 switch，根据不同的 action.type，返回不同的对象状态。
但是如果我们需要 type 很多，比如除了 visibilityFilter，还有 todos，难道要写一个长长的switch，当然不。
redux 提供一个 combineReducers 辅助函数，把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore。
我们把不同的 reducer 放在不同文件下。
``app/reducers/todo.js``
```
import {
  ADD_TODO,
  COMPLETE_TODO
} from '../actions';

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ];
    case COMPLETE_TODO:
      return [
        ...state.slice(0, action.index),
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ];
    default:
      return state
  }
}

export default todos;
```
然后通过一个 index.js 把他们合并。
``app/reducers/index.js``
```
import { combineReducers } from 'redux';
import todos from './todos';
import visibilityFilter from './visibilityFilter';

const rootReducer = combineReducers({
  todos,
  visibilityFilter
});

export default rootReducer;
```
## 展示组件
``app/components/AddTodo/index.jsx``
```
import React, { Component, PropTypes } from 'react';
import { findDOMNode } from 'react-dom';

export default class AddTodo extends Component {
  render() {
    return (
      <div>
        <input type='text' ref='input' />
        <button onClick={ e => this.handleClick(e) }>
          Add
        </button>
      </div>
    );
  }

  handleClick(e) {
    const inputNode = findDOMNode(this.refs.input);
    const text = inputNode.value.trim();
    this.props.onAddClick(text);
    inputNode.value = '';
  }
}

AddTodo.propTypes = {
  onAddClick: PropTypes.func.isRequired
};
```

``app/components/Todo/index.jsx``
```
import React, { Component, PropTypes } from 'react';

export default class Todo extends Component {
  render() {
    const { onClick, completed, text } = this.props;
    return (
      <li
        onClick={onClick}
        style={{
          textDecoration: completed ? 'line-through' : 'none',
          cursor: completed ? 'default' : 'pointer'
        }}
      >
        {text}
      </li>
    );
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
};
```

``app/components/TodoList/index.jsx``
```
import React, { Component, PropTypes } from 'react';
import Todo from '../Todo';

export default class TodoList extends Component {
  render() {
    return (
      <ul>
        {
          this.props.todos.map((todo, index) =>
            <Todo
              {...todo}
              onClick={() => this.props.onTodoClick(index)}
              key={index}
            />
          )
        }
      </ul>
    );
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
};
```

``app/components/Footer/index.jsx``
```
import React, { Component, PropTypes } from 'react';

export default class Footer extends Component {
  renderFilter(filter, name) {
    if(filter == this.props.filter) {
      return name;
    }
    return (
      <a
        href="#"
        onClick={e => {
          e.preventDefault();
          this.props.onFilterChange(filter);
        }}>
        {name}
      </a>
    );
  }

  render() {
    return (
      <p>
        SHOW
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    );
  }
}

Footer.propTypes = {
  onFilterChange: PropTypes.func.isRequired,
  filter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
};
```

可以看出，所有的展示组件需要的 state 和 数据，都从属性中获取的，所有的操作，都是通过容器组件给的回调函数来操作的。
他们尽可能地不拥有自己的状态，做无状态组件。

# end
关于 redux 的用法，这只是基础入门的部分，还有的多的搞基操作，比如异步数据流、Middleware、和 router 配合。
敬请期待～～～～
🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶🐶
