# Webpack和React小书

## 介绍
这本小书的目的是引导你进入 React 和 Webpack 的世界。他们两个都是非常有用的技术，如果同时使用他们，前端开发会更加有趣。

这本小书会提供所有相关的技能。如果你只是对 React 感兴趣，那可以跳过 Webpack 相关的内容，反之亦然。 如果想学习更多的相关知识可以移步 SurviveJS - Webpack and React。

### React

React 是一个能够让开发模块变成简单的库。一旦你理解他的工作原理，那你就可以用它搭建自己的程序，这是不同类似 Angular 那种试着包揽一切的框架不同的地方。

如果你想很快过一遍 React 的知识点，那么 React 官方教程 是一个很好的开始。

可能 React 最有趣的事是它一直会尝试调整传统的 web 组件的思路。它让我们重新思考关注点的分离。它（React Native）也会影响 App 开发。 React Native 提供了一种使用 Javascript 开发原生应用同时保证了原生性能。

### Webpack

Webpack 非常容易操作，它是一个模块合并的工具，本质就是一个能够把各种组件（HTML，CSS，JS）构建成项目。最方便的是你只需要初始化配置一次，Webpack 会替你做那些繁琐的事情，同时也保证了让你可以在项目中混合使用各种技术而不头疼。

如果你在 Webpack 方面完全是新手的，但想开始一个简单的教程的话，可以去 Pete Hunt's guide。你可以在那里学习到一些基础的使用，这里只是那边的一个补充。

## 介绍 Webpack

在 Web 开发历程上，我们构建了很多小型的技术解决方案，比如用 HTML 去描述页面结构，CSS 去描述页面样式，Javascript 去描述页面逻辑，或者你也可以用一些比如 Jade 去取代 HTML，用 Sass 或 Less 去取代CSS，用 CoffeeScript 或者 TypeScript 之类的去取代 Javascript，不过项目中的依赖可能是一件比较烦恼的事情。（需要安装额外很多的库）

这里有很多为什么我们需要尝试那些新技术的理由。不管我们用什么，总之，我们还是希望使用那些能够处理在浏览器端的方案，所以出来了编译方案。历史上已经有很多分享了，比如 Make 可能是很多解决方案中最知名且是可行的方案。Grunt 和 Gulp 是在是前端的世界中最流行的解决方案，他们两个都有很多非常有用的插件。NPM（Node.js 的包管理器）则包含了他们两个。

### Grunt

Grunt 是相比后面几个更早的项目，他依赖于各种插件的配置。这是一个很好的解决方案，但是请相信我，你不会想看到一个 300 行的 Gruntfile。如果你好奇 Grunt 的配置会如何，那么这里是有个从 Grunt 文档 的例子：

```
module.exports = function(grunt) {

  grunt.initConfig({
    jshint: {
      files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      options: {
        globals: {
          jQuery: true
        }
      }
    },
    watch: {
      files: ['<%= jshint.files %>'],
      tasks: ['jshint']
    }
  });

  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-watch');

  grunt.registerTask('default', ['jshint']);

};
```
### Gulp

Gulp 提供了一个不一样的解决方案，而不是依赖于各种插件的配置。Gulp 使用了一个文件流的概念。如果你熟悉 Unix，那么 Gulp 对你来说会差不多，Gulp 会提供你一些简单化的操作。在这个解决方案中，是去匹配一些文件然后操作（就是说和 JavaScript 相反）然后输出结果（比如输出在你设置的编译路径等）。这里有一个简单的 Gulpfile 的例子：

```
var gulp = require('gulp');
var coffee = require('gulp-coffee');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var del = require('del');

var paths = {
  scripts: ['client/js/**/*.coffee', '!client/external/**/*.coffee'],
};

// 不是所有的任务需要使用 streams
// 一个 gulpfile 只是另一个node的程序，所以你可以使用所有 npm 的包
gulp.task('clean', function(cb) {
  // 你可以用 `gulp.src` 来使用多重通配符模式
  del(['build'], cb);
});

gulp.task('scripts', ['clean'], function() {
  // 压缩和复制所有 JavaScript （除了第三方库）
  // 加上 sourcemaps
  return gulp.src(paths.scripts)
    .pipe(sourcemaps.init())
      .pipe(coffee())
      .pipe(uglify())
      .pipe(concat('all.min.js'))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest('build/js'));
});

// 监听文件修改
gulp.task('watch', function() {
  gulp.watch(paths.scripts, ['scripts']);
});

// 默认任务（就是你在命令行输入 `gulp` 时运行）
gulp.task('default', ['watch', 'scripts']);

```
这些配置都是代码，所以当你遇到问题也可以修改，你也可以使用已经存在的 Gulp 插件，但是你还是需要写一堆模板任务。

### Browserify

处理 Javascript 模块一直是一个大问题，因为这个语言在 ES6 之前没有这方面的概念。因此我们还是停留在90年代，各种解决方案，比如提出了 AMD。在实践中只使用 CommonJS （ Node.js 所采用的格式）会比较有帮助，而让工具去处理剩下的事情。它的优势是你可以发布到 NPM 上来避免重新发明轮子。

Browserify 解决了这个问题，它提供了一种可以把模块集合到一起的方式。你可以用 Gulp 调用它，此外有很多转换小工具可以让你更兼容的使用（比如 watchify 提供了一个文件监视器帮助你在开发过程中更加自动化地把文件合并起来），这样会省下很多精力。毋庸置疑，一定程度来讲，这是一个很好的解决方案。

### Webpack

Webpack 扩展了 CommonJs 的 require 的想法，比如你想在 CoffeeScript、Sass、Markdown 或者其他什么代码中 require 你想要的任何代码的话？那么 Webpack 正是做这方面的工作。它会通过配置来取出代码中的依赖，然后把他们通过加载器把代码兼容地输出到静态资源中。这里是一个 Webpack 官网 上的例子：
```
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```
在接下来的章节中我们会使用 Webpack 来构建项目来展示它的能力。你可以用其他工具和 Webpack 一起使用。它不会解决所有事情，只是解决一个打包的难题，无论如何，这是在开发过程中需要解决的问题。

## 启动

在开始之前，你需要把你的 Node.js 和 NPM 都更新到最新的版本。访问 nodejs.org 查看安装详情。我们将会使用 NPM 安装一些工具。
开始使用 Webpack 非常简单，我会展示给你看使用它的一个简单的项目。第一步，为你的项目新建一个文件夹，然后输入 npm init，然后填写相关问题。这样会为你创建了 package.json，不用担心填错，你可以之后修改它。

### 安装 Webpack

接下来我们安装 Webpack，我们要把它安装在本地，然后把它作为项目依赖保存下来。这样你可以在任何地方编译（服务端编译之类的）。输入 npm i wepack --save-dev。如果你想运行它，就输入 node_modules/.bin/webpack。

### 目录结构

项目的目录结构长这样：
```
/app

main.js
component.js
/build

bundle.js (自动创建)
index.html
package.json
webpack.config.js
```
我们会使用 Webpack 在我们的 /app 里来自动创建 bundle.js 。接下来，我们来设置

webpack.config.js。

### 设置 Webpack

Webpack 的配置文件长这样：

webpack.config.js

var path = require('path');


module.exports = {
    entry: path.resolve(__dirname, 'app/main.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js',
    },
};
运行你的第一个编译

现在我们有了一个最简单的配置，我们需要有什么东西去编译，让我们开始一个经典的 Hello World，设置 /app 像这样：

app/component.js

```
'use strict';


module.exports = function () {
    var element = document.createElement('h1');

    element.innerHTML = 'Hello world';

    return element;
};
```

app/main.js
```
'use strict';
var component = require('./component.js');


document.body.appendChild(component());
```

现在在你的命令行运行 webpack，然后你的应用会开始编译，一个 bundle.js 文件就这样出现在你的 /build 文件夹下，需要在 build/ 下的 index.html 去启动项目。

build/index.html

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8"/>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

这个文件可以用 html-webpack-plugin 来生成。如果你觉得冒险，那就把剩下的工具交给它来做。使用它就只有一个配置的问题。一般来说使用 Webpack 来工作就是这么个套路。
### 运行应用

只要双击 index.html 或者设置一个 Web 服务指向 build/ 文件夹。

### 设置 package.json scripts

npm 是一个非常好用的用来编译的指令，通过 npm 你可以不用去担心项目中使用了什么技术，你只要调用这个指令就可以了，只要你在 package.json 中设置 scripts 的值就可以了。

在这个案例中我们把编译步骤放到 npm run build 中是这样：

npm i webpack --save - 如果你想要把 Webpack 作为一个项目的开发依赖，就可以使用 --save-dev，这样就非常方便地让你在开发一个库的时候，不会依赖工具（但不是个好方法！）。
把下面的内容添加到 package.json中。
  "scripts": {
    "build": "webpack"
  }
现在你可以输入 npm run build 就可以编译了。

当项目越发复杂的时候，这样的方法会变得越来越有效。你可以把所有复杂的操作隐藏在 scripts 里面来保证界面的简洁。

不过潜在的问题是这种方法会导致如果你使用一些特殊的指令的时候只能在 Unix 环境中使用。所以如果你需要考虑一些未知的环境中的话，那么 gulp-webpack 会是一个好的解决方案。

注意 NPM 会找到 Webpack，npm run 会把他临时加到 PATH来让我们这个神奇的命令工作。
## 工作流

如果需要一直输入 npm run build 确实是一件非常无聊的事情，幸运的是，我们可以把让他安静的运行，让我们设置 webpack-dev-server。

### 设置 webpack-dev-server

第一步，输入 npm i webpack-dev-server --save，此外，我们需要去调整 package.json scripts 部分去包含这个指令，下面是基本的设置：

package.json

```
{
  "scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
  }
}
```

当你在命令行里运行 npm run dev 的时候他会执行 dev 属性里的值。这是这些指令的意思：

webpack-dev-server - 在 localhost:8080 建立一个 Web 服务器
--devtool eval - 为你的代码创建源地址。当有任何报错的时候可以让你更加精确地定位到文件和行号
--progress - 显示合并代码进度
--colors - Yay，命令行中显示颜色！
--content-base build - 指向设置的输出目录
总的来说，当你运行 npm run dev 的时候，会启动一个 Web 服务器，然后监听文件修改，然后自动重新合并你的代码。真的非常简洁！

访问 http://localhost:8080 你会看到效果。

## 浏览器自动刷新

当运行 webpack-dev-server 的时候，它会监听你的文件修改。当项目重新合并之后，会通知浏览器刷新。为了能够触发这样的行为，你需要把你的 index.html 放到 build/ 文件夹下，然后做这样的修改：

build/index.html

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8"/>
  </head>
  <body>
    <script src="http://localhost:8080/webpack-dev-server.js"></script>
    <script src="bundle.js"></script>
  </body>
</html>
```

我们需要增加一个脚本当发生改动的时候去自动刷新应用，你需要在配置中增加一个入口点。

```
var path = require('path');


module.exports = {
    entry: ['webpack/hot/dev-server', path.resolve(__dirname, 'app/main.js')],
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js',
    },
};
```
就是这样！现在你的应用就可以在文件修改之后自动刷新了。

### 默认环境

在上面的例子中我们创建了 index.html 文件来获取更多的自由和控制。同样也可以从 http://localhost:8080/webpack-dev-server/bundle 运行应用。这会触发一个默认的你不能控制的 index.html ，它同样会触发一个允许iFrame中显示重合并的过程。

## 引入文件

### 模块

Webpack 允许你使用不同的模块类型，但是 “底层”必须使用同一种实现。所有的模块可以直接在盒外运行。

- ES6 模块
```
import MyModule from './MyModule.js';
```
- CommonJS
```
var MyModule = require('./MyModule.js');
```
- AMD
```
define(['./MyModule.js'], function (MyModule) {

});
```
### 理解文件路径

一个模块需要用它的文件路径来加载，看一下下面的这个结构：
```
/app

/modules
MyModule.js
main.js (entry point)
utils.js
```
打开 main.js 然后可以通过下面两种方式引入 app/modules/MyModule.js

app/main.js

```
// ES6
import MyModule from './modules/MyModule.js';

// CommonJS
var MyModule = require('./modules/MyModule.js');
```

最开始的 ./ 是 “相对当前文件路径”

让我们打开 MyModule.js 然后引入 app/utils：

app/modules/MyModule.js

全选复制放进笔记
```
// ES6 相对路径
import utils from './../utils.js';

// ES6 绝对路径
import utils from '/utils.js';

// CommonJS 相对路径
var utils = require('./../utils.js');

// CommonJS 绝对路径
var utils = require('/utils.js');
```
相对路径是相对当前目录。绝对路径是相对入口文件，这个案例中是 main.js。

### 我需要使用文件后缀么？

不，你不需要去特意去使用 .js，但是如果你引入之后高亮会更好。你可能有一些 .js 文件和一些 .jsx 文件，甚至一些图片和 css 可以用 Webpack 引入，甚至可以直接引入 node_modules 里的代码和特殊文件。

记住，Webpack 只是一个模块合并！也就是说你可以设置他去加载任何你写的匹配，只要有一个加载器。我们稍后会继续深入这个话题。
