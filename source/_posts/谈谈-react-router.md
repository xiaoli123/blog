title: 谈谈 react-router
date: 2015-12-11 09:33:12
tags:
- 前端
- React
- react-router
categories: [React, react-router]
---
# 谈谈
最近使用的 React + webpack 来开发项目，感觉确实是爽的飞起，然而总感觉还是少了点什么。😫
😏对，是多页面，每次请求页面还要后端路由给你？😄多不爽啊，试试 react-router ，简直掌控一切的感觉，只开放一个页面路由接口，其他全给数据接口就👌了，😂可以和后端哥哥说拜拜了。
👋👋👋👋👋👋👋👋👋👋👋 啪啪啪啪啪～

![react-router](/img/react-router.png)

# 概述
先贴上官方文档 [https://github.com/rackt/react-router/tree/master/docs](https://github.com/rackt/react-router/tree/master/docs).
对了这里还有一份中文文档（不过不是很全）[http://react-guide.github.io/react-router-cn/](http://react-guide.github.io/react-router-cn/).
react-router 是 React 的完整前端路由解决方案，特别在做一个 spa 应用的时候，他能实现 url 和 视图ui 的同步，并且支持后端渲染，异步按需加载等等😄。
由于 react-router 文档的多变，这里的例子以当前版本 1.0.1 为准。（1.0之前文档每一个版本的变动都很大，索多了都是泪）

# 浏览器支持
所有支持 React 的浏览器。
倡议！建议大家也劝说身边的人用***现代化***浏览器，关爱前端开发者。

# 安装
```
npm install react-router@latest
```
同时，react-router 是基于 history 开发的，这里你需要安装 history。
注意 react-router 当前版本 1.0.1 依赖的是 history 1.13.1 请不要安装最新版。
不要问我为什么知道，被坑惨了；有同学问，那没有办法用 history 的最新版本嘛？毕竟这只是暂缓之计，解决方案还是有的，那就是等 react-router 作者解决咯 😂。
```
npm install history@1.13.1
```
构建工具的话，我依然建议是 webpack ， React 和 webpack 是一对好兄弟。
```
npm install webpack
```
webpack的使用方法可以看我的前两篇文章：
+ [webpack-best-practice-最佳实践-部署生产](http://qiutc.me/post/webpack-best-practice-%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5-%E9%83%A8%E7%BD%B2%E7%94%9F%E4%BA%A7.html)
+ [如何使用webpack—webpack-howto](http://qiutc.me/post/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8webpack%E2%80%94webpack-howto.html)
(求赞！！！)

# 小例子
```
// 加载依赖包，这是 es6 的语法（我好啰嗦）。
import React from 'react'
import { render } from 'react-dom'
// 这里从 react-router 引入了三个组件，先不解释。
import { Router, Route, Link } from 'react-router'

const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>
        {this.props.children}
      </div>
    )
  }
});

const Inbox = React.createClass({
  render() {
    return (
      <div>
        Inbox
      </div>
    )
  }
});

const About = React.createClass({
  render() {
    return (
      <div>
        About
      </div>
    )
  }
});

render((
  <Router>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
    </Route>
  </Router>
), document.getElementById('root'));
```
我就偷个懒把官方文档的demo直接copy了。
直接先来看 render 下面的内容，这里是用 jsx 语法。 最外层组件是 Router（可以把他看作是react-router提供的最外层容器） , 下一层是 Route，这个是路由组件。对应关系如下：
``/ ---> <App />``
``/about ---> <App><About /></App>``
``/inbox ---> <App><Inbox /></App>``
path 对应路径，component 对应当前路径渲染的组件。
Route 里面的 Route 表示在父组件路由的 path 路径下面的一级 path 对应的路由，这里的路由是父子嵌套，对应的组件也同样是父子嵌套的。
如果是多级嵌套也同样如此。

# 不想用 jsx 来写，换个方式吧～
以上例子可以改写成：
```
...
const routerConfig = [
  {
    path: '/',
    component: App,
    childrenRoutes: [
      { path: 'about', component: About },
      { path: 'inbox', component: Inbox },
    ]
  }
];
render((
  <Router routes={routeConfig} />
), document.getElementById('root'));
```
这里的结构就更清晰了，我是比较喜欢这种方式。

# 默认的路由
比如上面的例子，/ 对于的组件是 App , 如果 App 只是渲染了一个导航条，却没有自组件，那打开 比如 qiutc.me/ 的时候不是就没有内容了吗。
把上面例子改一下
```
...
// 添加组件 Index
const Index = React.createClass({
  render() {
    return (
      <div>
        Index
        Index
        Index
      </div>
    )
  }
});
// 修改配置
const routerConfig = [
  {
    path: '/',
    component: App,
    indexRoute: { component: Index },
    childrenRoutes: [
      { path: 'about', component: About },
      { path: 'inbox', component: Inbox },
    ]
  }
];

```
这里加了一个 ``indexRoute`` ，表示他在没有匹配子路由的时候，在 / 路由下渲染默认的子组件 Index。
路由组件嵌套也是一样的，比如：
```
      { path: 'about', component: About, indexRoute: {component: AboutIndex} },

```
以此类推。
# 404  NotFound
如果我们打开了一个没有设置路由的链接，就必然需要一个友好的 404 页面。配置如下：
```
...
// 添加 404 组件
const NotFound = React.createClass({
  render() {
    return (
      <div>
        404
        NotFound
      </div>
    )
  }
});
// 修改配置
const routerConfig = [
  {
    path: '/',
    component: App,
    indexRoute: { component: Index },
    childrenRoutes: [
      { path: 'about', component: About },
      { path: 'inbox', component: Inbox },
    ]
  },
  {
    path: '*',
    component: NotFound,
  }
];
```
如此简单。

# 绝对路径与重定向
```
const routerConfig = [
  {
    path: '/',
    component: App,
    indexRoute: { component: Index },
    childrenRoutes: [
      { path: 'about', component: About },
      {
        path: 'inbox',
        component: Inbox,
        childrenRoutes: [
          {
            path: 'message/:id',
            component: Message,
          }
        ],
      },
    ]
  },
  {
    path: '*',
    component: NotFound,
  }
];
```
在这里我们访问 /inbox/message/1 对于渲染 Message 组件，这个链接太长了，我们想直接 /message/1 那怎么办，改路由结构？太麻烦了！绝对路径可以帮你做到这个。
把 ``path: 'message/:id', `` 改为 ``path: '/message/:id', `` 就好了。
等等如果用户之前收藏的链接是 /inbox/message/1 ，那不是就打不开了嘛，和后端路由一样，react-router 也有重定向：redirect
```
const routerConfig = [
  {
    path: '/',
    component: App,
    indexRoute: { component: Index },
    childrenRoutes: [
      { path: 'about', component: About },
      {
        path: 'inbox',
        component: Inbox,
        childrenRoutes: [
          {
            path: '/message/:id',
            component: Message,
          },
          {
            path: 'message/:id',
            onEnter: function (nextState, replaceState) {
              replaceState(null, '/messages/' + nextState.params.id);
            }
          }
        ],
      },
    ]
  },
  {
    path: '*',
    component: NotFound,
  }
];
```
## onEnter 方法表示进入这个路由前执行的方法，在进入 /inbox/messages/:id 的前，执行
```
function (nextState, replaceState) {
  replaceState(null, '/messages/' + nextState.params.id);
}
```
``nextState``表示要进入的下一个路由，这里就是 /inbox/messages/:id ，``replaceState`` 表示替换路由状态的方法，把 /inbox/messages/:id 替换成 /messages/:id，然后就可以重定向到 /messages/:id。
## 同样的也有 onLeave 这个方法
表示在离开路由前执行。
———————————————————
当然如果你用的是 jsx 语法，有更简单的组件可以实现：
```
import { Redirect } from 'react-router'

React.render((
  <Router>
    <Route path="/" component={App}>
      <IndexRoute component={Index} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="/messages/:id" component={Message} />
        {/* 跳转 /inbox/messages/:id 到 /messages/:id */}
        <Redirect from="messages/:id" to="/messages/:id" />
      </Route>
    </Route>
  </Router>
), document.getElementById('root'))
```

# 路径匹配原理

## 嵌套关系

>React Router 使用路由嵌套的概念来让你定义 view 的嵌套集合，当一个给定的 URL 被调用时，整个集合中（命中的部分）都会被渲染。嵌套路由被描述成一种树形结构。React Router 会深度优先遍历整个理由配置来寻找一个与给定的 URL 相匹配的路由。

简单来讲，就是说，匹配的时候会先匹配到外层路径，然后依次遍历到内层。
比如 /inbox/messages/:id 会先匹配 /，渲染 / 对应的组件 App，然后再到 / 的下一层寻找 /inbox ，同样渲染 /inbox 对应的组件 Inbox，依次类推，直到到 message/:id。

tip：使用绝对路径可以忽略嵌套关系，如上面例子。

## 路径语法
路由路径是匹配一个（或一部分）URL 的 一个字符串模式。大部分的路由路径都可以直接按照字面量理解，除了以下几个特殊的符号：
- /:paramName – 匹配一段位于 /、? 或 # 之后的 URL。 命中的部分将被作为一个参数
- (/) – 在它内部的内容被认为是可选的
- /* – 匹配任意字符（非贪婪的）直到命中下一个字符或者整个 URL 的末尾，并创建一个 splat 参数

 ```
<Route path="/hello/:name">         // 匹配 /hello/michael 和 /hello/ryan
<Route path="/hello(/:name)">       // 匹配 /hello, /hello/michael 和 /hello/ryan
<Route path="/files/*.*">           // 匹配 /files/hello.jpg 和 /files/path/to/hello.jpg

 ```
## 优先级
最后，路由算法会根据定义的顺序自顶向下匹配路由。因此，当你拥有两个兄弟路由节点配置时，你必须确认前一个路由不会匹配后一个路由中的路径。例如：
```
<Route path="/comments" ... />
<Redirect from="/comments" ... />
```
第二个是不会被执行的。

# 拿到参数路径的
比如上面的 /messages/:id ，这个id可能是我们在 Message 获取数据时需要的 id。
他会被当做一个参数传给 params，parmas 会传给 Message 组件的 props：
```
const Message = React.createClass({
    render: function() {
      return (
        <div>{ this.props.params.id }</div>
      );
    }
});
```
这样就可以获取到了。

# history 配置
React Router 是建立在 history 之上的。 简而言之，一个 history 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 location 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。
常用的 history 有三种形式， 但是你也可以使用 React Router 实现自定义的 history。

- createHashHistory
- createBrowserHistory
- createMemoryHistory

## 这三个有什么区别呢：

### createHashHistory
这是一个你会获取到的默认 history ，如果你不指定某个 history 。它用到的是 URL 中的 hash（#）部分去创建形如 example.com/#/some/path 的路由。
这个 支持 ie8＋ 的浏览器，但是因为是 hash 值，所以不推荐使用。

### createBrowserHistory
Browser history 是由 React Router 创建浏览器应用推荐的 history。它使用 History API 在浏览器中被创建用于处理 URL，新建一个像这样真实的 URL example.com/some/path。

### Memoryhistory
不会在地址栏被操作或读取。

## 使用
```
import { createBrowserHistory, useBasename } from 'history';
const historyConfig = useBasename(createHistory)({
  basename: '/'        // 根目录名
});
...
render((
  <Router routes={routeConfig} History={historyConfig} />
), document.getElementById('root'));
```

# Link&IndexLink
## Link
我们在最开头看到这样一个东西：
```
const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>
        {this.props.children}
      </div>
    )
  }
});
```
Link 会被渲染成 a ，to 其实就是 href ，
但是 react-router 会阻止默认跳转页面，而改成 history 路由的变换。

参数：
- to
切换到的路由地址
- query
跟在 url 的 query 参数，比如
```
query={{q: "que"}} 对应 `/example?a=que
```
这里的 query 同样可以像 params 会被传入下一个路由组件的 props
- hash
跟在 url 的 hash 参数，比如
```
hash={111} 对应 `/example#111
```
这里的 query 同样可以像 params 会被传入下一个路由组件的 props
- activeClassName
当前 url 路径如果和 Link 的 to 匹配 这个 Link 就会有一个定义的属性，比如：
```
在 /index 下
<Link to="/index" activeClassName={"active"} activeStyle={{color: 'red'}} >/</Link>
这里渲染出来的 a 标签会有一个激活的 active 类名，还会有一个颜色 red
<Link to="/about" activeClassName={"active"} activeStyle={{color: 'red'}} >/</Link>
这里渲染出来的 a 标签就不会有以上属性
```
- activeStyle
同上
- onClick
点击的时候执行的函数，会传入一个 e 事件对象，你可以 ``e.stopPropagation()`` 阻止默认路由切换。

## IndexLink
在上面有一个问题如果：
```
在 / 下 和 /index
<Link to="/" activeClassName={"active"} activeStyle={{color: 'red'}} >/</Link>
这个 Link 渲染出来的 a 标签都会激活 active 属性，并且会带上 color: 'red'
因为 / 和 /index 和 ／ 都是匹配的
```
这时候就可以用:
```
<IndexLink to="/" activeClassName={"active"} activeStyle={{color: 'red'}} >/</IndexLink>
只会在 / 下呗激活，在 /index 或者其他下面，不会被激活
```

# 未完待续
关于 ___根据路由按需异步加载js___ 和 ___服务器端渲染路由视图___ 以及 ___react-router的更高级用法___ 会在下一篇文章来探讨。
毕竟哥也需要去深入研究一下才敢献丑。😂😂😂
🐶
