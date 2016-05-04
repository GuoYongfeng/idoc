<h1 style="font-size: 40px;text-align:center;color: #007cdc;">使用Webpack搭建生产环境工作流</h1>

## css文件单独加载

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

## 将js文件的应用和第三方分开打包

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


## 给文件添加hash

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

## 异步加载（实现资源加载的性能优化）

虽然CommonJS是同步加载的，但是webpack也提供了异步加载的方式。这对于单页应用中使用的客户端路由非常有用。当真正路由到了某个页面的时候，它的代码才会被加载下来。

指定你要异步加载的 **拆分点**。看下面的例子

```js
if (window.location.pathname === '/feed') {
  showLoadingState();
  require.ensure([], function() { // 这个语法痕奇怪，但是还是可以起作用的
    hideLoadingState();
    require('./feed').show(); // 当这个函数被调用的时候，此模块是一定已经被同步加载下来了
  });
} else if (window.location.pathname === '/profile') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./profile').show();
  });
}
```

剩下的事就可以交给webpack，它会为你生成并加载这些额外的 **chunk** 文件。

webpack 默认会从项目的根目录下引入这些chunk文件。你也可以通过 `output.publicPath`来配置chunk文件的引入路径

```js
// webpack.config.js
output: {
    path: "/home/proj/public/assets", // webpack的build路径
    publicPath: "/assets/" // 你require的路径
}
```
