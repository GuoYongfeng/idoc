<h1 style="font-size: 40px;text-align:center;color: blue;">增强Webpack配置，让开发赢在起点</h1>

> 花点时间熟悉webpack的配置，可以让开发的工作更加轻松。

## 1 实现代码热替换

在webpack.config.js更新一下入口文件配置即可实现编辑器中保存代码就可在浏览器中实现刷新的效果，棒棒哒。
代码清单：`webpack.config.js`
```
entry: [
      'webpack-dev-server/client?http://localhost:8080',
      path.resolve(__dirname, 'app/index.js')
]
```

顺便说一句，如果你是gulp的使用者，推荐结合`browser-sync`的reload接口和gulp的watch功能结合，也可以很轻松的实现这样的功能。

## 2 使用react-hot-loader实现组件级的hot reload

虽然实现了代码的热替换，只要在编辑器中保存我们编辑的代码，浏览器即可实时刷新。但同时也有一个烦恼，如果我们的项目开发中用到了几十个组件，为了测试某个组件我们需要一步步操作到固定的步骤去实现，一旦保存编辑器中修改的一行代码，从入口文件开始的所有代码都全部刷新了一次，这样很不利于调试。

幸运的是，我们有react-hot-loader，接下来我们去实现它，先下载下来：
```
$ npm install --save-dev react-hot-loader
```

然后重新配置webpack.config.js。
代码清单：`webpack.config.js`

```
var path = require('path');
var webpack = require('webpack');

module.exports = {
    entry: [
      'webpack/hot/dev-server',
      'webpack-dev-server/client?http://localhost:8080',
      path.resolve(__dirname, 'app/index.js')
    ],
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'bundle.js',
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loaders: ['react-hot', 'babel'],
          exclude: path.resolve(__dirname, 'node_modules')
        }
      ]
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin(),
      new webpack.NoErrorsPlugin()
    ]
};

```

这里新增了react-hot-loader去解析React组件，同时加上了热替换的插件HotModuleReplacementPlugin和防止报错的插件NoErrorsPlugin，如果这里不用HotModuleReplacementPlugin这个插件也可以，只需要在webpack-dev-server运行的时候加上--hot这个参数即可，都是一样的。

更多资料请参考[这里](http://gaearon.github.io/react-hot-loader/getstarted/)

## 3 加载并解析你的样式文件

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

接下来测试一下，在app目录新增一个基本的React组件，然后在index.js中引用这个组件。

新增一个目录coponents并且在目录下新建一个Button组件
```
$ cd app && mkdir components
$ cd components && mkdir Button
$ cd Button && touch Button.js Button.less
```

代码清单：`app/components/Button.js`
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

代码清单：`app/components/Button.less`
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

代码清单：`app/index.js`
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

## 4 css文件单独加载

通过上面的例子，css文件的引入、解析、运行已经跑通，BUT，目前我们的css文件全部被打包在bundle.js一个文件里面。这可不是一件好事，后续代码量一上来，文件越来越胖，我想老板一定会抓你去做性能优化的，所以，我们需要把css文件单独打包出来。

extract-text-webpack-plugin插件可以帮助我们解决这个问题，现在让我们先来下载。
```
$ npm install extract-text-webpack-plugin --save-dev
```

然后配置webpack。
代码清单：`webpack.config.js`
```
var path = require('path');
var webpack = require('webpack');
var ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
    entry: [
      'webpack/hot/dev-server',
      'webpack-dev-server/client?http://localhost:8080',
      path.resolve(__dirname, 'app/index.js')
    ],
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'bundle.js',
    },
    resolve: {
      extension: ['', '.js', '.jsx', '.json']
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loaders: ['react-hot', 'babel'],
          exclude: path.resolve(__dirname, 'node_modules')
        },
        {
          test: /\.css/,
          loader: ExtractTextPlugin.extract("style-loader", "css-loader")
        },
        {
          test: /\.less/,
          loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
        }
      ]
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin(),
      new webpack.NoErrorsPlugin(),
      new ExtractTextPlugin("bundle.css")
    ]
};
```

同时，也需要在我们的index.html去引入bundle.css文件。
代码清单：`public/index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React Demo</title>
  <link rel="stylesheet" href="bundle.css">
</head>
<body>
  <div id="app"></div>
  <script src="bundle.js"></script>
</body>
</html>
```

执行`npm run dev`后你就可以在浏览器中看到加载的分离后的css文件了。

另外这里手动去修改index.html是一个不是很友好的体验，这里暂且按下不表，后续我们会通过插件来统一生成public下的资源，这样让调试和部署更加便捷。

## 5 图片资源的加载

图片资源的加载相对简单，代码的写法都可以就近依赖require，通过url-loader来解析加载。先进行下载：
```
$ npm install --save-dev url-loader
```
进行webpack配置。
代码清单：`webpack.config.js`
```
loaders: [
  {
    test: /\.(png|jpg)$/,
    loader: 'url-loader?limit=8192'
  }
]
```

这里我们匹配了png和jpg两种格式的图片使用url-loader进行加载，比如：
```
.logo {
  background-image: url('logo.png');
}
```
并且设置了limit参数值是8192，意思是匹配到小于8K（8 * 2014=8192）的图片时将其进行base64转化后放入文件中，大于8k的图片则进行单独加载。

咱们需要做一个测试，来跑通刚才的配置，新建一个container目录用于存放容器级（可以粗略的理解为页面级）组件，新建一个static目录用于存放图片、图标字体、音频视频等资源，我们下载两张图片放入用于后续的代码引用。

```
$ cd app && mkdir container static
$ cd container && touch App.js App.less
```

同时我们将调整index.js中的代码，将index.js只加载App容器组件，App加载组件Button。我们来看下新的代码。

代码清单：`app/index.js`
```
'use strict';
import React from 'react';
import ReactDOM from 'react-dom';
import App from './container/App';

let root = document.getElementById('app');
ReactDOM.render( <App />, root );
```
代码清单：`app/container/App.js`
```
import React, { Component } from 'react';
import Button from '../components/Button/Button';

import './App.less';

class App extends Component {
  render(){
    return (
      <div className="app">
        <Button />
        <div className="tip"></div>
      </div>
    );
  }
}

export default App;

```
代码清单：`app/container/App.less`
```
.app {
  width: 200px;
  height: 300px;
  // 这是一张19K的jpg图片
  background-image: url(../static/angelababy.jpg);
  position: relative;
  .tip {
    width: 100px;
    height: 80px;
    // 这是一张2k的png图片
    background-image: url(../static/tip.png);
    position: absolute;
    right: -100px;
  }
}
```

执行`npm run dev`跑一次代码，正常展示后我们可以看到控制台的信息，2k的图片被base64，19k的图片正常加载。

## 6 图标字体的加载

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
代码清单：`app/container/App.js`
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

## 7 将js文件的应用和第三方分开打包

修改webpack配置中的entry入口，并且添加CommonsChunkPlugin插件抽取出第三方资源。

代码清单：`webpack.config.js`
```
entry: {
 index: [
    'webpack/hot/dev-server',
    'webpack-dev-server/client?http://localhost:8080',
    path.resolve(__dirname, 'app/index.js')
  ],
  vendor: ['react', 'react-dom']
},
plugins: [
   new webpack.HotModuleReplacementPlugin(),
   new webpack.NoErrorsPlugin(),
   new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.js'),
   new ExtractTextPlugin("bundle.css")
 ]
```

同时我们还有修改index.html文件的引用。
代码清单：`public/index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React Demo</title>
  <link rel="stylesheet" href="bundle.css">
</head>
<body>
  <div id="app"></div>
  <script src="vendor.js"></script>
  <script src="index.js"></script>
</body>
</html>

```

## 8 加上一个小小的调试工具

我们在配置中新增devtool字段，并设置值为source-map，这样我们就可以在浏览器中直接调试我们的源码，在控制台的sources下，点开可以看到`webpack://`目录，点开有惊喜哦。

代码清单：`webpack.config.js`
```
devtool: 'cheap-module-source-map'
```

## 9 将html也进行统一产出

前面我们还是先在public目录手动加上的index.html，这样在项目中不是很适用，因为我们希望public产出的资源应该是通过工具来统一产出并发布上线，这样质量和工程化角度来思考是更合适的。下面我们来实现。

在app目录下新建一个index.html文件，并写上简单的代码。
```
$ cd app && touch index.html
```
代码清单：`app/index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React课堂demo</title>
</head>
<body>
  <div id="app"></div>
</body>
</html>
```

接下来需要下载一个webpack的插件html-webpack-plugin。
```
$ npm install --save-dev html-webpack-plugin
```

修改webpack配置。
代码清单：`webpack.config.js`
```
// 引用这个plugin
var HtmlWebpackPlugin = require('html-webpack-plugin');

// 这里省略其他配置代码

plugins: [
	  // 使用这个plugin，这是最简单的一个配置，更多资料可到github查看
      new HtmlWebpackPlugin({
        title: 'zhufeng-react',
        template: './app/index.html',
      })
]
```

ok，运行`npm run dev`跑一遍，效果正常。


## 10 添加文件的hash

我们的开发的产品最终是要上线的，添加文件hash可以解决由于缓存带来的问题，所以我们需要试着给文件加上hash。其实很简单，在文件的后面加上`?[hash]`就行，当然，这也是简单的写法。

照例贴着到这个阶段的配置代码吧。
```
var path = require('path');
var webpack = require('webpack');
var ExtractTextPlugin = require("extract-text-webpack-plugin");
var publicPath = path.resolve(__dirname, 'public');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: {
      index: [
        'webpack/hot/dev-server',
        'webpack-dev-server/client?http://localhost:8080',
        path.resolve(__dirname, 'app/index.js')
      ],
      vendor: ['react', 'react-dom']
    },
    output: {
        path: publicPath,
        filename: '[name].js?[hash]'
    },
    resolve: {
      extension: ['', '.js', '.jsx', '.json']
    },
    module: {
      loaders: [
        {
          test: /\.js$/,
          loaders: ['react-hot', 'babel'],
          exclude: path.resolve(__dirname, 'node_modules')
        },
        {
          test: /\.css/,
          loader: ExtractTextPlugin.extract("style-loader", "css-loader")
        },
        {
          test: /\.less/,
          loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
        },
        {
          test: /\.(png|jpg)$/,
          loader: 'url?limit=8192'
        },
        {
          test: /\.(woff|woff2|ttf|svg|eot)(\?v=\d+\.\d+\.\d+)?$/,
          loader: "url?limit=10000"
        }
      ]
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin(),
      new webpack.NoErrorsPlugin(),
      new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.js?[hash]'),
      new ExtractTextPlugin("[name].css?[hash]", {
          allChunks: true,
          disable: false
      }),
      new HtmlWebpackPlugin({
        title: 'zhufeng-react',
        template: './app/index.html',
      })
    ],
    devtool: 'cheap-module-source-map'
};

```

11. 区分环境的标识

项目中有些代码我们只为在开发环境（例如日志）或者是内部测试环境（例如那些没有发布的新功能）中使用，那就需要引入下面这些魔法全局变量（magic globals）：

```js
if (__DEV__) {
  console.warn('Extra logging');
}
// ...
if (__PRERELEASE__) {
  showSecretFeature();
}
```

同时还要在webpack.config.js中配置这些变量，使得webpack能够识别他们。

```js
// webpack.config.js

// definePlugin 会把定义的string 变量插入到Js代码中。
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

配置完成后，就可以使用 `BUILD_DEV=1 BUILD_PRERELEASE=1 webpack`来打包代码了。
值得注意的是，`webpack -p` 会删除所有无作用代码，也就是说那些包裹在这些全局变量下的代码块都会被删除，这样就能保证这些代码不会因发布上线而泄露。

## 结语

这些webpack可以让我们初期的开发游刃有余，但是实际项目开发的时候，需要增添很多功能，比如开发环境和生产环境的不同配置；打包的优化配置；让运行时的解析更快；配合测试框架...
