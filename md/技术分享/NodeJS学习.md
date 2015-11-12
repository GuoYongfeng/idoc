# NodeJS学习

![node](/img/node.png)

## 0 介绍

虽然介绍性的话题往往都比较没营养，但是对于初接触Nodejs的同学或是只是听说过Nodejs的同学而言，有必要郑重其事地来说一下什么是Nodejs。

> JS本身是脚本语言，脚本语言都需要一个解析器才能运行。对于写在HTML页面里的JS，浏览器充当了解析器的角色。
> 而对于需要独立运行的JS，NodeJS就是一个解析器，是JS语言的服务器运行环境，可以使JS流畅的运行于服务器端。

而以上提到的**运行环境**，我们可以这样理解：
1. 首先要避免一个误区，如今有无数的前端框架和类库都是各种`.js`结尾，虽然Node.js也是这样，但请不要混淆概念。
2. JavaScript就好比是找到了一个宿主环境，可以通过Node在服务器端运行。
3. Node提供大量工具模块、丰富的第三方包以及基于node的web框架，使得JavaScript语言可以实现文件的读写、进程管理以及网络通信等，在这个意义上，Node又是JavaScript的工具库。
4. 而且，Node内部采用Google公司的V8引擎，作为JavaScript语言解释器；通过自行开发的libuv库，调用操作系统资源。

好了，至少我们现在对Nodejs有了基本认识。

## 1 为什么要学nodejs

**它很火**，`npmjs.org`社区的活跃程度可以直接访问网站即可看到，大量的第三方包，伸手党的天堂。

**社区欣欣向荣**

**github**，node相关的项目 10万个

**很强**，使用node做的产品：
  - node-webkit等基于node的桌面开发
  - jade和ejs等模板引擎
  - grunt和gulp等构建工具
  - express和koa等web框架

> 其他语言能做的事情，nodejs都可以做，甚至可以做的更好，我想，对于一个前端工程师而言，我实在是找不出一个不学nodejs的理由。

**推荐关注**

[github](https://github.com/)在这里可以找到大量nodejs相关的项目，阅读源码源码，查看新技术的一手资料

[nodejs官网](https://nodejs.org)关注Node版本更新，包括api功能及使用、bug修复、新增特性以及未来的发展趋势

[npm官网](https://www.npmjs.com/)在这里搜索你想用的包，参考别人的源代码

[stackoverflow](http://stackoverflow.com/)问答社区，有什么疑惑直接在这开问吧，会有很多热情的好基友来帮你解答问题的，比如服务异常、配置什么的。

## 2 安装

### 2.1 windows上安装

- 先去下载一下git的客户端，可以运行git bash，方便使用shell命令

step1. 进入nodejs官方网站下载软件(nodejs.org),
step2. 下载完成后，双击默认安装。安装程序会自动添加环境变量
step3. 检测nodejs是否安装成功。打开cmd命令行 输入 :
```
node - v
```
step4. 检查npm是否安装。由于新版的NodeJS已经集成了npm，所以之前npm也一并安装好了。同样可以使用cmd命令行进行确认。
```
npm -v
```

### 2.2 mac上安装

- 升级系统
- 升级xcode
```
xcode-select -p
xcode-select --install
```
- 安装Homebrew
前提是python和ruby安装好
官网查看方法
- 安装
```
brew install node
```
- 检查
```
node -v
```

### 2.3 linux上安装

- 先要扫平环境问题
也可以到官网查看
要求gcc 4.2+和g++ 4.2+以及python 2.6和gnu的版本要求
- 检查
```
cat /etc/redhat-release
rpm -q gcc rpm -q gcc-c++
yum -y install gcc gcc-c++ kernel-devel
```
ubuntu下可以apt-get
```
cd /usr/src
wget 链接
tar -xf node包名
cd node包
./config
make
sudo make install
```
- 检查安装是否成功
```
node -v
npm -v
```
### 2.4 版本号管理和说明

> 目前最新的都已经到`5.0.0`，这是从0.12版本后，nodejs和iojs合并了，直接从4.0版本开始发展了，到如今已经是5.0版本，当然，还在持续更新迭代中。

版本不同，接口API都是不一样的 要关注

关于nodejs版本号的说明

偶尔位的版本是稳定版本，而一般奇数位的就是非稳定版本，这几乎是在业界大家都达成共识了。比如0.6.x就是稳定版本，而0.11.x就是新功能测试的非稳定版本

建议选择最新的稳定版本进行使用

```
npm install -g n
n 5.0.0
n
上下选择
```

- nvm
- nvvm

## 3 node初体验


### 3.1 在命令行中直接感受

在浏览器中控制台，我们可以直接编写js代码进行运行

在你的shell窗口，键入node后回车，即可进入node的repl环境运行js代码

### 3.2 起个web服务器

//server.js
```
var http = require('http')
http.createServer(function(req, res){
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello world");
  }).listen(1337, "127.0.0.1");

```

是的，就是这样神奇，短短几行代码，就创建了一个web服务，而且，请不要轻视它，这还是一个高性能的web服务器，在某些

### 3.3 commonjs代码规范说明

也许你对上面的代码有所好奇，所以我们就来简单分析下以上代码。

首先，Node程序它是由许多个模块组成的，每个模块就是一个文件。Node模块采用了CommonJS规范。

根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在一个文件定义的变量（还包括函数和类），都是私有的，对其他文件是不可见的。

而且，CommonJS规定，每个文件的对外接口是module.exports对象。这个对象的所有属性和方法，都可以被其他文件导入。

如下，我们来定义一个模块
```javascript
var x = 5;

var addX = function(value) {
  return value + x;
};

// 导出变量和方法
module.exports.x = x;
module.exports.addX = addX;
```
理解：上面代码通过module.exports对象，定义对外接口，输出变量x和函数addX。module.exports对象是可以被其他文件导入的，它其实就是文件内部与外部通信的桥梁。

好，继续，我们来引用刚才定义的模块
```javascript
var example = require('./example.js');

console.log(example.x); // 5
console.log(addX(1)); // 6
```

### 3.3.1 module对象
每个模块内部，都有一个module对象，代表当前模块。它有以下属性。
```
module.id 模块的识别符，通常是带有绝对路径的模块文件名。
module.filename 模块的文件名，带有绝对路径。
module.loaded 返回一个布尔值，表示模块是否已经完成加载。
module.parent 返回一个对象，表示调用该模块的模块。
module.children 返回一个数组，表示该模块要用到的其他模块。
```
- module.exports属性
module.exports属性表示当前模块对外输出的接口，其他文件加载该模块，实际上就是读取module.exports变量。

- exports变量
为了方便，Node为每个模块提供一个exports变量，指向module.exports。这等同在每个模块头部，有一行这样的命令。
### 3.3.2 require命令的解读
Node.js使用CommonJS模块规范，内置的require命令用于加载模块文件。

require命令的基本功能是，读入并执行一个JavaScript文件，然后返回该模块的exports对象。如果没有发现指定模块，会报错。

```
// example.js
var invisible = function () {
  console.log("invisible");
}

exports.message = "hi";

exports.say = function () {
  console.log(message);
}
```

### 3.3.3 require加载模块的规则
require命令用于加载文件，后缀名默认为.js，比如：
```
var foo = require('foo');
//  等同于
var foo = require('foo.js');
```

举例来说，脚本/home/user/projects/foo.js执行了require('bar.js')命令，Node会依次搜索以下文件。
```
/usr/local/lib/node/bar.js
/home/user/projects/node_modules/bar.js
/home/user/node_modules/bar.js
/home/node_modules/bar.js
/node_modules/bar.js
```
这样设计的目的是，使得不同的模块可以将所依赖的模块本地化。

如果指定的模块文件没有发现，Node会尝试为文件名添加.js、.json、.node后，再去搜索。.js件会以文本格式的JavaScript脚本文件解析，.json文件会以JSON格式的文本文件解析，.node文件会以编译后的二进制文件解析。

## 4. 模块和包

基于以上的学习，我们大体理解了node是什么，而且会基本的node程序的书写，现在来进一步理解模块和包，进入学习node的快车道。

### 4.1 概念区分

在node里面，单个的文件即模块，每个模块可以提供很多个API接口进行使用。

而包的概念，比模块大，多个模块可以封装成一个包（甚至一个也行），且包的封装和发布，使用和更新，都需要遵循包的相关机制。包的管理可以使用npm，npm是最大的包管理和分发平台。

这里提到的npm有两层含义：
1. Node.js的开放式模块登记和管理系统。
2. Node.js默认的模块管理器，是一个命令行下的软件，用来安装和管理node模块。

### 4.2 模块分类
模块和文件是对应的
- 核心模块fs http path
- 文件模块
```
var myfile = require('./myfile.js');
```
- 第三方模块
```
var gulp = require('gulp');
```

### 4.3 包的概念和npm的配置

npm不需要单独安装。在安装node的时候，会连带一起安装npm。但是，node附带的npm可能不是最新版本，最好用下面的命令，更新到最新版本。

step1. npm作为一个NodeJS的模块管理，我们要先配置npm的全局模块的存放路径以及cache的路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立**node_global**及**node_cache**两个文件夹。我们就在cmd中键入两行命令：
```
npm config set prefix "D:\Program Files\nodejs\node_global"
npm config set cache "D:\Program Files\nodejs\node_cache"
```
step2. 下面这一步非常关键，我们需要设置系统变量。进入我的电脑→属性→高级→环境变量。
```
在系统变量下新建“NODE_PATH”
输入内容“D:\Program Files\nodejs\node_global\node_modules”
```
step3. 安装bower或是gulp等常用工具
```
npm install bower gulp -g
```
这个时候，你在命令行就可以全局的使用安装的工具了，可以命令行中直接确认
```
gulp -v
bower -v
```
step4. 接下来我们可以进一步学习如何用npm发布自己的包
```
// 初始化一个package
npm init
// 添加你的包的管理用户
npm adduser
// 发布你的包
npm publish --tag 0.1.0
```

### 4.3 学习封装一个包

学习包的封装和发布
示例: 封装一个可以读取命令行的工具

```
#! /usr/bin/env node
```

- 创建一个模块
- 导出模块
- 加载模块
- 使用模块

示例

## 5 node中的全局对象

### buffer对象

### process对象

### __dirname和__filename

### stdin和stdout

### console对象

## 6 学习node中的内置模块和对象


### 6.1 Url模块

url和uri的区别
url是uri的一个子集
url有自己的属性规则，编码

```
parse
format
resolve
```

进入node的repl环境
示例演示和讲解

### 6.2 Path模块

字符串解析

### 6.3 Fs模块

> 示例：遍历目录，操作文件，以前抓取数据的示例

### 6.4 Http模块

- http：起个服务
- https
- url设计
- query string

> 示例：

### 6.5 Cluster模块

- process
- cluster

### 6.6 Events模块和异步编程

- promise
- 异常处理
- 处理回调深渊

## 7 使用express做个示例

### 7.1 express简介

基于 Node.js 平台，快速、开放、极简的 web 开发框架。

### 7.2 安装

```
npm install express express-generator -g
```

### 7.3 模板jade

### 7.4 写示例代码

### 7.5 优化
