title: 如何使用webpack—webpack-howto
date: 2015-10-21 10:58:10
tags:
- webpack
- 翻译
- JavaScript
categories: [webpack]
---
# 如何使用webpack
本文章翻译自petehunt的webpack-howto教程，[原地址https://github.com/petehunt/webpack-howto](https://github.com/petehunt/webpack-howto)，还有源码。

## 本教程的目标
这是一本如何使用webpack来处理代码的“烹饪书”，它包括了我们在Instagram用的大多数东西以及我们还没在项目中用到过的。
我（作者）的建议：把这本书作为你学习webpack文档的开端，然后配合看官方文档的说明。

## 预备知识
* 你了解browserify，ReqiureJs或者其他类似的东西。
* 你明白这些东西的价值：
    * 打包分离
    * 异步加载
    * 打包静态资源，比如图片和CSS

## 1.为什么使用webpack？
* __他类似browserify__ 但是可以把你的应用分离成许多文件，如果你有许多页面在你的单页应用里面，用户只需要下载当前页面所需要的代码。如果你跳转到另一个页面，他们不需要重新加载通用的代码。
* __他能替代grunt或者gulp大部分的功能__ 因为他可以构建和打包CSS，预处理CSS，编译JS和打包处理图片，甚至更多事情。
他支持AMD和CommonJs以及其他的模块化系统（Angular，ES6）。如果你不知道用什么，那么可以用CommonJs。

## 2.给用browserify的人
下面的代码是等价的：
```
browserify main.js > bundle.js
```
```
webpack main.js bundle.js
```
然而，webpack比browserify更加强大，你可能会更愿意去编写一个`` webpack.config.js ``来合理地组织你的项目：
```
// webpack.config.js
module.exports = {
    entry: './main.js',
    output: {
        filename: 'bundle.js'
    }
};
```
这就是JS，所以你可以很轻松地在这个文件里编写代码。

## 3.如何调用webpack？
切换带包含webpack.config.js的目录然后在命令行运行：
* ``webpack`` 执行一次开发时的编译
* ``webpack -p`` 执行一次生成环境的编译（压缩）
* ``webpack --watch`` 在开发时持续监控增量编译（很快）
* ``webpack -d`` 让他生成SourceMaps

## 编译JS语言
webpack和browserify一样能转化（transforms）的RequireJs插件是一个加载器（loader）
你能让webpack加载CoffeeScript和Facebook的JSX+ES6支持（你必须先 ``npm install babel-loader coffee-loader``）:
```
// webpack.config.js
module.exports = {
    entry: './main.js',
    output: {
        filename: 'bundle,js'
    },
    module: {
        loader: [
            { test: /\.coffee$/, loader: 'coffee-loader' },
            { test: /\.js$/, loader: 'bebel-loader' }
        ]
    }
};
```
为了开启后缀名的自动补全，你需要添加``resolve.extensions``参数来指定哪些文件是webpack需要搜索的：
```
// webpack.config.js
module.exports = {
    entry: './main.js',
    output: {
        filename: 'bundle,js'
    },
    module: {
        loader: [
            { test: /\.coffee$/, loader: 'coffee-loader' },
            { test: /\.js$/, loader: 'bebel-loader' }
        ]
    },
    resolve: {
        // 你现在可以使用 ``require('file')`` 来代替 ``require('file.coffee')`` 。
        extensions: ['', '.js', '.json', '.coffee', '.jsx']
    }
};
```

## 5.样式和图片
首先你需要更新你的代码去请求 ``require()`` 你的静态资源（命名和他们在node里面使用 ``require()`` 一样）。
```
require('./bootstrap.css');
require('./myapp.less');
var img = document.createElement('img');
img.src = require('./glyph.png');
```
当你请求CSS（或者less或者其他）的时候，webpack会把CSS像字符串一样内联到JS包里面，require（）会在页面插入一个style标签。
当你请求图片时，webpack会内联一个图片的URL链接到你的代码包然后通过``require()``返回。
但是你需要去教webpack去做这件事（再次重复，用loaders）：
```
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    path: './build', // 图片和js会被打包到这个目录
    publicPath: 'http://mycdn.com/', // 这个是用来生成图片地址的URL
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.less$/, loader: 'style-loader!css-loader!less-loader' }, // 使用 ! 来链接多个loader
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      {test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // 内联小于8k的base64图片，其他的直接使用URL
    ]
  }
};
```

## 6.功能标志
有些代码可能我们只想要在生产环境下运行（比如日志logging）和我们内部的测试调试服务器（就像我们的测试人员在测试应用），可以在你的代码引入全局环境变量。
```
if(__DEV__) {
    console.warn('Extra logging');
}
// ...
if(__PRERELEASE__) {
    showSecretFeature();
}
```
这会教webpack这些奇特的全局环境：
```
// webpack.config.js

// definePlugin 接收字符串插入到代码当中, 所以你需要的话可以写上 JS 的字符串.
var definePlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
  __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  plugins: [definePlugin]
};
```
你可以在控制台里执行 ``BUILD_DEV=1 BUILD_PRERELEASE=1 webpack`` .
注意：``webpack -p``会丑化并且剔除死代码，然和包含在里面的代码都会被剔除掉，这样你就不用担心泄露秘密咯~

## 7.多重入口
比如说你有一个profile和一个feed页面，你不想让用户在只想要profile页面的时候下载feed页面的代码，所以你可以创建多个包，每个页面都有一个"main module"。
```
// webpack.config.js
module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // 模板基于上面入口文件的key
  }
};
```
在profile页面，插入 ``<script src="build/Profile.js"></script>`` .feed页面也会这样。

## 8.优化通用代码
feed和profile页面共享许多通用的代码（比如React和公用样式和组件）。
webpack可以计算出哪些代码是它们公用的然后生成一个共享的包然后在页面之间被缓存。
```
// webpack.config.js

var webpack = require('webpack');

var commonsPlugin =
  new webpack.optimize.CommonsChunkPlugin('common.js');

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // Template based on keys in entry above
  },
  plugins: [commonsPlugin]
};
```
在script标签添加之前添加 ``<script src="build/common.js"></script>`` 就可以使用缓存啦~

## 9.异步加载
CommonJs是同步加载的，但是webpack提供了一个方法可以异步加载指定的依赖项。
这对于客户端的路由是很有用的，每一个页面都需要路由，但是在某个功能被用到之前你不会想去下载这部分代码的。
在你想要异步加载的地方指定分离点，比如：
```
if (window.location.pathname === '/feed') {
  showLoadingState();
  require.ensure([], function() { // 这个语法很怪异但是他能执行
    hideLoadingState();
    require('./feed').show(); // 当这个函数被调用的时候, 这个模块时可以同步的。
  });
} else if (window.location.pathname === '/profile') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./profile').show();
  });
}
```
webpack会做剩下的事情并且生成额外的包然后给你加载。
webpack在html的script标签加载时会假设这些文件是在你的根目录下的，比如，你可以使用 ``output.publicPath来配置他``
```
// webpack.config.js
output: {
    path: "/home/proj/public/assets", //path指向你编译时的文件目录
    publicPath: "/assets/" //publicPath指向你引用文件是考虑的地址
}
```

# 其他资源
可以看一看一个成功的团队如何在真实项目中使用webpack：[http://youtu.be/VkTCL6Nqm6Y](http://youtu.be/VkTCL6Nqm6Y)
这是Pete Hunt 在OSCon大会上说的关于webpack在Instagram.com中的使用。

# FAQ
__webpack不像模块化？__
webpack非常的模块化。他能让插件自身集成到编译过程中的许多地方相比于browserify和requireJs.
许多看起来是在被编写在核心代码中的功能实际上是被默认加载的插件，他们可以被覆盖。
