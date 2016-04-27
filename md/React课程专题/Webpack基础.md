<h1 style="font-size: 40px;text-align:center;color: #007cdc;">Webpack基础</h1>

webpack是一款强大的模块加载器兼打包工具，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。

<img src="/img/webpack/what-is-webpack.png" />


`webpack能做什么`
- 把依赖树拆成可按需加载的块
- 让初始化加载时间尽可能地少
- 每个静态资源都是一个模块
- 模块化集成第三方库
- 尽可能地自定义打包器的每一部分

`webpack有什么特别之处`
- 做代码拆分
- 丰富的loaders
- 出色的插件系统
- 支持第三方库的解析

接下来我们将一步步熟悉Webpack的使用，并使用它来搭建一套前端工作流。

## 初始化项目

创建一个项目
```
$ mkdir webpack-demos && cd webpack-demos
$ git init
$ touch README.md .gitignore
$ npm init
```

编辑.gitignore
```
node_modules
*.log*
.idea
```

下载webpack和webpack-dev-server
```
$ npm install --save-dev webpack webpack-dev-server
# 同时进行全局安装，进行测试，但后面会将命令配置在package.json里面
$ npm install -g webpack webpack-dev-server
```

创建webpack的配置文件
```
$ touch webpack.config.js
```

进行基本配置
```
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, 'src/index.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js'
    },
};

```
建立src和build两个目录
```
$ mkdir src build
$ cd src && touch index.js component.js
$ cd ../build && touch index.html
```

- src 目录存放源码
- build 目录存放编译打包之后的资源


```
/* src/index.js */
var component = require('./component.js');

component();


```

```
/* src/component.js */
module.exports = function(){
  alert('component');
}



```

```
/* build/index.html */
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Webpack demos</title>
</head>
<body>
  <div id="app"></div>
  <script src="bundle.js"></script>
</body>
</html>

```

执行webpack命令
```
$ webpack
```

可以看到控制台出现如下信息：
```
Hash: cf7cc9272c664f542fcb
Version: webpack 1.13.0
Time: 80ms
    Asset     Size  Chunks             Chunk Names
bundle.js  2.04 kB       0  [emitted]  main
   [0] ./src/index.js 60 bytes {0} [built]
   [1] ./src/component.js 57 bytes {0} [built]
    + 1 hidden modules

```

build目录下也新增了一个bundle.js文件

## webpack和webpack-dev-server的基本命令

```
$ webpack --help
```
执行以上命令，可以在控制台看到很多webpack相关的命令，选取几个常用的介绍下。

* `webpack` 开发环境下编译
* `webpack -p` 产品编译及压缩
* `webpack --watch` 开发环境下持续的监听文件变动来进行编译(非常快!)
* `webpack -d` 引入 source maps
* `webpack --progress` 显示构建进度
* `webpack --display-error-details` 这个很有用，显示打包过程中的出错信息


另外，让我们使用webpack-dev-server来起一个本地服务进行调试：
```
$ webpack-dev-server --progress --colors --content-base build
```

打开`localhost:8080`，回车即可。

那么执行webpack-dev-server后面的几个参数是什么意思呢？
- `webpack-dev-server` - 在 localhost:8080 建立一个 Web 服务器
- `webpack-dev-server --devtool eval` - 为你的代码创建源地址。当有任何报错的时候可以让你更加精确地定位到文件和行号
- `webpack-dev-server --progress` - 显示合并代码进度
- `webpack-dev-server --colors` - 命令行中显示颜色！
- `webpack-dev-server --content-base build` - 指向设置的输出目录
- `webpack-dev-server --inline` 可以自动加上dev-server的管理代码，实现热更新
- `webpack-dev-server --hot` 开启代码热替换，可以加上HotModuleReplacementPlugin
- `webpack-dev-server --port 3000` 设置服务端口

## 多文件入口

```
$ cd src && touch entry1.js entry2.js
```

```
/* webpack.config.js */
var path = require('path');

module.exports = {
    entry: {
      entry1: './src/entry1.js',
      entry2: './src/entry2.js',
    },
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: '[name].js'
    },
};
```

## 使用Babel-loader来解析es6和jsx

我们在src/index.js里面尝试写一个最基本的组件代码，暂时不用理会代码为什么要这么写，这里先把ES6语法和JSX语法加进来，用于跑通我们的开发环境，后续会有专题内容来详细讲述。

代码清单：`src/index.js`
```
'use strict';

import React, { Component } from 'react';
import ReactDOM from 'react-dom';

class HelloWorld extends Component {
  render(){
    return (
      <h1>Hello world</h1>
    )
  }
}

ReactDOM.render(<HelloWorld />, document.getElementById('app'));
```

ok，我们看到，我们的代码用到了基本的react.js和react-dom.js，而且使用的是ES6的语法来封装的组件和应用模块。

所以接下来我们要做两件事：
1. 下载相应的模块：
```
$ npm install --save react react-dom
```
2. 下载并配置babel，以解析ES6语法和JSX语法。

babel是一款强大的解析器，拥有活跃而且完善的生态，不仅可以做JS相关的各种语法的解析，还提供丰富的插件功能。

```

$ npm install babel-loader babel-core --save-dev
```

安装后我们需要配置webapck.config.js文件

代码清单：`webpack.config.js`
```
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, 'src/index.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js',
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loader: 'babel-loader'
        }
      ]
    }
};
```

这里指定了使用babel-loader来解析js文件，但是并没有告诉babel应该如何来解析，所以我们需要创建一个babelrc配置文件

```
$ touch .babelrc
```

然后编辑babelrc
代码清单：`.babelrc`
```
{
  "presets": ["es2015", "react", "stage-0"],
  "plugins": []
}
```

为什么配置的是这两个参数，解释一下，配置的preset字段是在为babel解析做预设，告诉babel需要使用相关的预设插件来解析代码，plugins字段，顾名思义，就是用来配置使用babel相关的插件的，这里暂且按下不表。

这里使用到了三个预设需要下载安装
```
$ npm install --save-dev babel-preset-es2015 babel-preset-react babel-preset-stage-0
// 其中stage-0预设是用来说明解析ES7其中一个阶段语法提案的转码规则
```

到这里我们可以重新来运行一次webpack以查看效果，为方便运行，我们把webpack-dev-server的相关代码放到package.json的script里面去

代码清单：`package.json`

```
"scripts": {
    "dev": "./node_modules/.bin/webpack-dev-server --progress --colors --content-base build"
}
```

好了，开始运行试一下吧

```
$ npm run dev
```

在浏览器中访问：`http://localhost:8080/`


## devServer

刚才我们看到，在运行webpack-dev-server的时候，后面带了一串参数，这里我们可以使用devServer字段统一在webpack.config.js文件里面维护。

```
/* webpack.config.js */
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, 'src/index.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js',
    },
    devServer: {
      publicPath: "/static/",
      stats: { colors: true },
      port: 8080,
      contentBase: 'build',
      inline: true
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loader: 'babel-loader'
        }
      ]
    }
};

```

同时，我们可以简化scripts字段的配置了
```
"scripts": {
    "dev": "./node_modules/.bin/webpack-dev-server"
}
```

对应的修改index.html文件中的资源引用地址
```
<script src="/static/bundle.js"></script>
```

ok, npm run dev即可

更多请看[这里](http://webpack.github.io/docs/webpack-dev-server.html)

## resolve

resolve下常用的是extension和alias字段的配置：
- extension 不用在require或是import的时候加文件后缀
```
/* webpack.config.js */
resolve: {
    extensions: ["", ".js", ".jsx", ".css", ".json"],
},
```

```
/* src/component.js */

export default () => {
  alert('component');
}

```

```
/* src/index.js */
'use strict';

import React, { Component } from 'react';
import ReactDOM from 'react-dom';

import component from './component';

component();

class HelloWorld extends Component {
  render(){
    return (
      <h1>Hello world</h1>
    )
  }
}

ReactDOM.render(<HelloWorld />, document.getElementById('app'));

```

刷新看到运行效果，而此时import的时候直接没有写文件后缀。

- alias 配置别名，加快webpack构建的速度

```
var path = require('path');

var pathToReact = path.join(__dirname, "./node_modules/react/dist/react.js");
var pathToReactDOM = path.join(__dirname, "./node_modules/react-dom/dist/react-dom.js");

module.exports = {
    entry: path.resolve(__dirname, 'src/index.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js',
    },
    devServer: {
      publicPath: "/static/",
      stats: { colors: true },
      port: 3000,
      contentBase: 'build',
      inline: true
    },
    resolve: {
      extensions: ["", ".js", ".jsx", ".css", ".json"],
      alias: {
        'react': pathToReact,
        'react-dom': pathToReactDOM
      }
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loader: 'babel-loader'
        }
      ],
      noParse: [pathToReact, pathToReactDOM]
    }
};

```

我们这里做了两件事情：
1.每当 "react" 在代码中被引入，它会使用压缩后的 React JS 文件，而不是到 node_modules 中找。
2.每当 Webpack 尝试去解析那个压缩后的文件，我们阻止它，因为这不必要。

## 解析样式文件
style-loader css-loader 样式内嵌
css module 配置?modules
less-loader；
## devtools
## 图标字体等资源
file-loader
