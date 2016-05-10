title: "webpack 2 的新特性 - What's new in webpack 2"
date: 2016-02-02 17:31:32
tags:
- webpack
- JavaScript
categories: [webpack]
---
![webpack](/img/webpack.png)

原文 [What's new in webpack 2](https://gist.github.com/sokra/27b24881210b56bbaff7)

# webpack 2 中的新特性
创造永无止境，现在 webpack 2 的版本已经更新到了 2.0.5-beta+ 。

# 主要的变化
## ES6 Modules
webpack 2 已经支持原生的 ES6 的模块加载器了，这意味着 webpack 2 能够理解和处理 ``import``和``export``了，而不用把他们转化成 CommonJS 来处理了。
```
import { currentPage, readPage } from "./book";

currentPage === 0;
readPage();
currentPage === 1;
```
```
// book.js
export var currentPage = 0;

export function readPage() {
    currentPage++;
}

export default "This is a book";
```

### 用 ES6 来做代码拆分
ES6 的模块加载器定义了``System.import``这一个方法，``System.import``能够在运行时动态加载 ES6 模块。
webpack 把``System.import``线程作为一个交叉指针然后把被要求的模块放到一个分离的代码块里面。（这里真心看不懂，乱来了😂，反正意思就是那样）
``System.import``函数把模块名作为参数，然后返回一个异步回调函数。
```
function onClick() {
    System.import("./module").then(module => {
        module.default;
    }).catch(err => {
        console.log("Chunk loading failed");
    });
}
```
好消息：模块加载失败可以被捕捉到错误了。

### 动态表达（异步请求js）
可以可以把部分代码传递给 ``System.import`` ，它的操作其实和 CommonJS 是类似的，给所有可能的文件创建一个环境。意思就是说当你传递那部分代码的模块还不确定的时候，webpack 会自动生成所有可能的模块，然后根据需求加载。
这个特性在前端路由的时候很有用，可以实现按需加载资源。
```
function route(path, query) {
    return System.import("./routes/" + path + "/route")
        .then(route => new route.Route(query));
}
// This creates a separate chunk for each possible route
```

### 混合使用 ES6 和 AMD 和 CommonJS (Mixing ES6 with AMD and CommonJS)
在 AMD 和 CommonJS 模块加载器中，你可以混合使用所有（三种）的模块类型（即使是在同一个文件里面）。在下面的情况中 webpack 的处理时类似的。
```
// CommonJS consuming ES6 Module
var book = require("./book");

book.currentPage;
book.readPage();
book.default === "This is a book";
```
```
// ES6 Module consuming CommonJS
import fs from "fs"; // module.exports map to default
import { readFileSync } from "fs"; // named exports are read from returned object+

typeof fs.readFileSync === "function";
typeof readFileSync === "function";
```

### babel 和 webpack
``es2015`` balel 的默认预处理会把 ES6 模块加载器转化成 CommonJS 模块加载。要是想使用 webpack 新增的对原生 ES6 模块加载器的支持，你需要使用 ``es2015-webpack`` 来代替。

### 对 ES6 优化细述
ES6 底层支持一些新的优化。比如在很多情况下它可以查明哪些 exports 是使用过的，哪些是未使用过的。
很多情况下，当 webpack 可以肯定地认为一个 export 它没有使用过，他会忽略暴漏 export 给其他模块的的声明，这样可以极小成本地标志一个 export 未使用过并且忽略它。
<br/>
在以下的情况中很可能查明使用状况：
- named import
- default import
- reexport

在以下的情况中不大可能查明使用状况：
- import * as ...
- CommonJS or AMD consuming ES6 module
- System.import

## 结构配置
在过去，由于环境的不同需要去处理不同环境下的结构配置，webpack 2 带来了一种新的方式去配置。
配置文件可以暴漏一个返回结构配置的函数，这个函数被 CLI 调用，然后值通过 ``--env`` 传递给结构配置的函数。
你可以传递一个字符串（--env dev => "dev"）或者一个复杂的配置对象 (--env.minimize --env.server localhost => {minimize: true, server: "localhost"}) 给这个函数。我推荐大家使用对象，因为它更具有可扩展性，当是这取决于你。
例子：
```
// webpack.config.babel.js
exports default function(options) {
    return {
        // ...
        devtool: options.dev ? "cheap-module-eval-source-map" : "hidden-source-map"
    };
}
```

## 配置项
这里是一些主要的重构[https://github.com/webpack/enhanced-resolve](https://github.com/webpack/enhanced-resolve)。它意味着配置项也改变了。
大多数是简化了配置，或者是让配置更不容易出错。
新的配置：
```
{
    modules: [path.resolve(__dirname, "app"), "node_modules"]
    // (was split into `root`, `modulesDirectories` and `fallback` in the old options)
    // In which folders the resolver look for modules
    // relative paths are looked up in every parent folder (like node_modules)
    // absolute paths are looked up directly
    // the order is respected

    descriptionFiles: ["package.json", "bower.json"],
    // These JSON files are read in directories

    mainFields: ["main", "browser"],
    // These fields in the description files are looked up when trying to resolve the package directory

    mainFiles: ["index"]
    // These files are tried when trying to resolve a directory

    aliasFields: ["browser"],
    // These fields in the description files offer aliasing in this package
    // The content of these fields is an object where requests to a key are mapped to the corresponding value

    extensions: [".js", ".json"],
    // These extensions are tried when resolving a file

    enforceExtension: false,
    // If false it will also try to use no extension from above

    moduleExtensions: ["-loader"],
    // These extensions are tried when resolving a module

    enforceModuleExtension: false,
    // If false it's also try to use no module extension from above

    alias: {
        jquery: path.resolve(__dirname, "vendor/jquery-2.0.0.js")
    }
    // These aliasing is used when trying to resolve a module
}
```

# 一些大变化
## Promise polyfill

## Loaders 的配置
在 loaders 的配置中使用了 ``resourcePath`` 来替代原来的 ``resource``
```
loaders: [
    {
        test: /\.css$/,
        loaders: [
            "style-loader",
            { loader: "css-loader", query: { modules: true } },
            {
                loader: "sass-loader",
                query: {
                    includePaths: [
                        path.resolve(__dirname, "some-folder")
                    ]
                }
            }
        ]
    }
]
```
## Minimize
在 UglifyJsPlugin 中不再支持让 Loaders 最小化文件的模式了，``debug`` 配置也被移除了，Loaders 不能从 webpack 的配置中读取到他们的配置项了，取代的方案是：
```
new webpack.LoaderOptionsPlugin({
    test: /\.css$/, // optionally pass test, include and exclude, default affects all loaders
    minimize: true,
    debug: false,
    options: {
        // pass stuff to the loader
    }
})
```
