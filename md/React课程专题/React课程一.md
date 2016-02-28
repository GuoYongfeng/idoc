
<h1 style="font-size: 40px;text-align:center;color: blue;">搭建基本环境，跑通示例代码</h1>

> 准备环境，并且跑通基本示例代码

## 1 介绍

### 1.1 环境配置

- 编辑器：Atom / Sublime Text 3 / Webstorm
- Chrome浏览器调试
- 安装Nodejs全局环境 && npm
- 基本命令行环境，windows用户可以使用Git Bash || CMD命令行窗口

### 1.2 基本知识储备

- React相关的基本思想
- 阅读过ES6/ES7相关的语法
- 对新型构建工具webpack有所了解
- 知道Babel是干什么用的
- 会基本的git操作命令

## 2 创建项目

```
$ mkdir zhufeng-react && cd zhufeng-react
$ git init
$ touch .gitignore
```

现在我们新建了一个项目目录并且使用git管理起来了，需要编辑一下.gitignore来忽略掉我们不想管理的一些文件

代码清单：`.gitignore`
```
node_modules/
.idea/
.project
*.log*
```

我们在用npm来初始化项目
```
$ npm init
```

并且创建两个基本的工程目录
```
$ mkdir app public
```

给项目添加基本代码

代码清单：`app/index.js`
```
alert('hello world!');
```

代码清单：`public/index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React Demo</title>
</head>
<body>
  <div id="app"></div>
  <script src="./bundle.js"></script>
</body>
</html>

```

## 3 安装webpack和webpack-dev-server

```
// 全局安装，方便使用命令行
$ npm install webpack webapck-dev-server -g
// 为项目安装依赖，后续需要require使用相关的API
$ npm install webpack webpack-dev-server --save-dev
```

### 3.1 初步配置webpack.config.js

```
$ touch webpack.config.js
```

代码清单：`webpack.config.js`
```
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, 'app/index.js'),
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'bundle.js',
    }
};
```

okay，让我们将服务启动，看到基本效果。

```
$ webpack-dev-server --progress --colors --content-base public
```

### 3.2 基于React + ES6的基本代码体验

我们在app/index.js里面尝试写一个最基本的组件代码，暂时不用理会代码为什么要这么写，这里先把ES6语法和JSX语法加进来，用于跑通我们的开发环境，后续会有专题内容来详细讲述。

代码清单：`app/index.js`
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

ok，我们看到，我们的代码用到了基本的react.js和react-dom.js两个模块，而且使用的是ES6的语法来封装的组件和应用模块。

所以接下来我们要做两件事：
1. 下载相应的模块：
```
$ npm install --save react react-dom
```
2. 下载并配置babel，以解析ES6语法和JSX语法。

## 4 安装和配置babel

babel是一款强大的解析器，拥有活跃而且完善的生态，不仅可以做JS相关的各种语法的解析，还提供丰富的插件功能。

```
// 先全局安装babel-cli以方便运行babel命令和babel-node命令
$ npm install babel-cli -g
$ npm install babel-loader babel-core --save-dev
```

安装后我们需要配置webapck.config.js文件

代码清单：`webpack.config.js`
```
var path = require('path');

module.exports = {
    entry: path.resolve(__dirname, 'app/index.js'),
    output: {
        path: path.resolve(__dirname, 'public'),
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
    "dev": "webpack-dev-server --progress --colors --content-base public"
}
```

好了，开始运行试一下吧

```
$ npm run dev
```

在浏览器中访问：`http://localhost:8080/`
