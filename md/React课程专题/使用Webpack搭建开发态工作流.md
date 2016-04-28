<h1 style="font-size: 40px;text-align:center;color: #007cdc;">使用Webpack搭建开发态工作流</h1>

接上一篇：[Webpack基础](/idoc/html/React课程专题/Webpack基础.html)，跑完基础流程后我们可以继续使用Webpack搭建我们的开发态的工作流。

## 1.Hot Module Replacement  HMR热更新

实现热更新有几种方式，也是容易让大家混淆的。

### 方案1：在webpack配置

对每个模块livereload，生产环境使用，依据代码更新情况只下载需要更新的部分

- 在webpack配置

```
entry: [
  'webpack/hot/dev-server',
  'webpack-dev-server/client?http://localhost:8080',
  path.resolve(__dirname, 'src/index.js')
],

...

plugins: [
  new webpack.HotModuleReplacementPlugin(),
  new webpack.NoErrorsPlugin()
]

```

如果想自己刷新页面，webpack配置改为
```
webpack/hot/only-dev-server
```

### 方案2：通过webpack-dev-server开启热更新

- webpack-dev-server启动参数

```
webpack-dev-server --hot
// 如果--hot选项会开启代码热替换，可以加上
// HotModuleReplacementPlugin，这样的话代码中可以不用写HotModuleReplacementPlugin

webpack-dev-server --inline
// 可以自动加上dev-server的管理代码，这样不用在entry的地方加上webpack/hot/dev-server
```

- 在webpack配置

```
entry: [
  'webpack-dev-server/client?http://localhost:8080',
  path.resolve(__dirname, 'src/index.js')
]
```

## 2.React-hot-loader 组件级热更新

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

## 3.HTML Webpack Plugin 解析html模板


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


## 4.open-browser-webpack-plugin 自动打开浏览器

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


## 5.区分环境标识 Environment flags


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

## 6.Code splitting 代码拆分 bundle-loader

## 7.Exposing global variables 暴露全局对象
