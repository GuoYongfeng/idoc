<h1 style="font-size: 40px;text-align:center;color: #007cdc;">Webpack基础</h1>

webpack是一款强大的模块加载器兼打包工具，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。

<img src="/img/webpack/what-is-webpack.png" />

**webpack有什么特别之处**

- 插件plugins：rich plugins & flexible
- 构建性能Performance: async I/O
- 加载器Loaders: bundle any static resource
- 对模块的支持Support: supports AMD and CommonJs module styles & support most existing libraries.
- 代码拆分Code Splitting: split your codebase into chunks
- 性能优化Optimizations: can do many optimizations to reduce the output size & handle request caching
- 开发者工具Development Tools: SourceMaps & development middleware & development server
- 多端适用Multiple targets: web & node.js

接下来我们将一步步熟悉Webpack的使用，并使用它来搭建一套前端工作流。


## 1.初始化项目

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

## 2.webpack和webpack-dev-server的基本命令

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
* `webpack --profile` 输出性能数据，可以看到每一步的耗时

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
- `webpack-dev-server --content-base build` - 设置静态资源的输出目录（Using this config webpack-dev-server will serve the static files in your build folder. ）
- `webpack-dev-server --inline` 可以自动加上dev-server的管理代码，实现热更新
- `webpack-dev-server --hot` 开启代码热替换，可以加上HotModuleReplacementPlugin
- `webpack-dev-server --port 3000` 设置服务端口

> 关于webpack-dev-server的简单介绍：webpack-dev-server是一个小型的node.js Express服务器,它使用webpack-dev-middleware中间件来为通过webpack打包生成的资源文件提供Web服务。它还有一个通过Socket.IO连接着webpack-dev-server服务器的小型运行时程序。webpack-dev-server发送关于编译状态的消息到客户端，客户端根据消息作出响应。


## 3.多文件入口

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

## 4.使用Babel-loader来解析es6和jsx

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


## 5.devServer

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

## 6.resolve

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

## 7.解析样式文件
style-loader css-loader 样式内嵌
css module 配置?modules
less-loader；


前面的大部分工作都在处理JS逻辑的解析和加载，但是我们还一直没有提我们的样式文件应该如何去处理。

我们在课程中暂且约定使用less预处理器（saas的类似）来写我们项目的样式，那么首先需要下载几个webpack的loader

```
$ npm install --save-dev style-loader css-loader less-loader
```

进行webpack配置。
代码清单：`webpack.config.js`
```
loaders: [
	{
      test: /\.js$/,
      loaders: ['react-hot', 'babel'],
      exclude: path.resolve(__dirname, 'node_modules')
    },
    {
      test: /\.css/,
      loader: 'style!css'
    },
    {
      test: /\.less/,
      loader: 'style!css!less'
    }
]
```

接下来测试一下，在src目录新增一个基本的React组件，然后在index.js中引用这个组件。

新增一个目录coponents并且在目录下新建一个Button组件
```
$ cd src && mkdir components
$ cd components && mkdir Button
$ cd Button && touch Button.js Button.less
```

代码清单：`src/components/Button.js`
```
import React, { Component } from 'react';

class Button extends Component {
  handleClick(){
    alert('戳我干嘛！');
  }
  render(){
    const style = require('./Button.less');

    return (
      <button className="my-button" onClick={this.handleClick.bind(this)}>
        快戳我
      </button>
    );
  }
}

export default Button;

```

代码清单：`src/components/Button.less`
```
.my-button {
  color: #fff;
  background-color: #2db7f5;
  border-color: #2db7f5;
  padding: 4px 15px 5px 15px;
  font-size: 14px;
  border-radius: 6px;
}
```

代码清单：`src/index.js`
```
'use strict';
import React from 'react';
import ReactDOM from 'react-dom';
import Button from './components/Button/Button';

let root = document.getElementById('app');
ReactDOM.render( <Button />, root );
```

搞定，跑一把。
```
$ npm run dev
```
修改一下less文件，浏览器会自动刷新，DONE，看起来还是很不错的样子。

## 8.devtool


我们在配置中新增devtool字段，并设置值为source-map，这样我们就可以在浏览器中直接调试我们的源码，在控制台的sources下，点开可以看到`webpack://`目录，点开有惊喜哦。

代码清单：`webpack.config.js`
```
devtool: 'cheap-module-source-map'
```

devtool可以有几个配置项：

|devtool|	build speed	| rebuild speed	|production supported |quality|
|---|---|---|---|---|
|eval|	+++	|+++	|no	|generated code|
|cheap-eval-source-map	|+|	++|	no|	transformed code (lines only)|
|cheap-source-map|	+	|o	|yes|	transformed code (lines only)|
|cheap-module-eval-source-map|	o	|++|	no|	original source (lines only)|
|cheap-module-source-map	|o|	-	|yes|	original source (lines only)|
|eval-source-map|	–	|+	|no|	original source|
|source-map|	–|	–	|yes	|original source|

## 9.图标字体等资源

图标字体的加载可以选择file-loader 或 url-loader 进行加载，配置如下（示例配置，大家在项目中最好还是按实际情况配置）
```
{
  test: /\.(woff|woff2|ttf|svg|eot)(\?v=\d+\.\d+\.\d+)?$/,
  loader: "url?limit=10000"
}
```

更多loader可以参考[webpack wiki](https://github.com/webpack/docs/wiki/list-of-loaders)。

照例，我们需要把配置跑通，首先下载我们熟悉的bootstrap，它给我们提供了整套的css并且还有优秀的图标字体库。

```
$ npm install bootstrap --save
```

在App.js里面应用bootstrap。
代码清单：`src/container/App.js`
```
import React, { Component } from 'react';
import Button from '../components/Button/Button';

import 'bootstrap/dist/css/bootstrap.css';
import './App.less';

class App extends Component {
  render(){
    return (
      <div className="text-center">
        <Button />
        <div className="tip"></div>
        {/* 这里我们使用以下图标字体 */}
        <span className="glyphicon glyphicon-asterisk"></span>
      </div>
    );
  }
}

export default App;
```

跑一下代码，一切正常，有没有感觉webpack果然是前端开发神器。

## 未完待续

请看下一篇：使用webpack搭建开发态工作流
