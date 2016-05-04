<h1 style="font-size: 40px;text-align:center;color: #007cdc;">使用Webpack搭建开发态工作流</h1>

接上一篇：[Webpack基础](/idoc/html/React课程专题/Webpack基础.html)，跑完基础流程后我们可以继续使用Webpack搭建我们的开发态的工作流。

## 自动刷新

webpack-dev-server有两种模式支持自动刷新——iframe模式和inline模式。

- 在iframe模式下：页面是嵌套在一个iframe下的，在代码发生改动的时候，这个iframe会重新加载；使用iframe模式无需额外的配置，只需在浏览器输入以下地址：

```
http://localhost:8080/webpack-dev-server/index.html
```
- 在inline模式下：一个小型的webpack-dev-server客户端会作为入口文件打包，这个客户端会在后端代码改变的时候刷新页面。

// 1.启动webpack-dev-server的时候带上inline参数
```
webpack-dev-server --inline
```

// 2.给HTML插入JS
```
<script src="http://localhost:3000/webpack-dev-server.js"></script>
```

// 3.webpack配置
```
entry: [
  'webpack-dev-server/client?http://localhost:3000',
  path.resolve(__dirname, 'src/index.js')
],
```

## Hot Module Replacement 模块热替换

webpac-dev-server支持Hot Module Replacement，即模块热替换，在前端代码变动的时候无需整个刷新页面，只把变化的部分替换掉。使用HMR功能也有两种方式：命令行方式和Node.js API。

// 1.cli命令行方式
```
webpack-dev-server --inline --hot
```

// 2.Node.js API方式
```
entry: [
  'webpack/hot/dev-server',
  path.resolve(__dirname, 'src/index.js')
],
devServer: {
  hot: true
},
plugins: [
  new webpack.HotModuleReplacementPlugin(),
]

```

更多内容可以参见这篇文章：http://www.jianshu.com/p/941bfaf13be1，这篇文章将webpack官网上对webpack-dev-server的代码刷新部分的介绍进行了翻译。

## React-hot-loader 组件级热更新

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

## HTML Webpack Plugin 解析html模板


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


## open-browser-webpack-plugin 自动打开浏览器

这个相对很简单，下载open-browser-webpack-plugin这个插件，配置在webpack里面，等我们的资源构建完成之后，自动打开浏览器，提高开发体验。

```
$ npm install open-browser-webpack-plugin --save-dev
```

然后在webpack里面进行配置
```
var openBrowserWebpackPlugin = require('open-browser-webpack-plugin');

......

plugin: [
  new openBrowserWebpackPlugin({ url: 'http://localhost:8080' })
]
```


## 区分环境标识 Environment flags


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

在package.json里面配置运行脚本：
```
"scripts": {
    "publish-mac": "export NODE_ENV=prod && webpack-dev-server -p --progress --colors",
    "publish-win": "set NODE_ENV=prod && webpack-dev-server -p --progress --colors"
}
````

值得注意的是，`webpack -p` 会删除所有无作用代码，也就是说那些包裹在这些全局变量下的代码块都会被删除，这样就能保证这些代码不会因发布上线而泄露。

## Exposing global variables 暴露全局对象

如果想将report数据上报组件放到全局，有两种办法：

方法一：

在loader里使expose将report暴露到全局，然后就可以直接使用report进行上报

```
{
    test: path.join(config.path.src, '/js/common/report'),
    loader: 'expose?report'
},
```

方法二：


如果想用R直接代表report，除了要用expose loader之外，还需要用ProvidePlugin帮助，指向report，这样在代码中直接用R.tdw， R.monitor这样就可以

```
new webpack.ProvidePlugin({
    "R": "report",
}),
```
