<h1 style="text-align:center;color:blue;font-size:44px;">Node.js知识学习</h1>

<img width="840" src="/img/node.png" />

# Chapter 1 -- Node.js入门知识

> 导读：在这一部分的内容里，主要会简单介绍一下什么是Node.js，它有哪些用武之地，我们为什么要去学习它。另外，还会简单的讨论下Nodejs的优缺点以及适用场景。

## 基本介绍

**Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 的包管理器 npm，是全球最大的开源库生态系统。**

以上提到的**运行环境**，我们可以这样理解：
> JS本身是脚本语言，脚本语言都需要一个解析器才能运行。对于写在HTML页面里的JS，浏览器充当了解析器的角色。
> 而对于需要独立运行的JS，NodeJS就是一个解析器，是JS语言的服务器运行环境，使其流畅的运行于服务器端。

**Nodejs的特点是：**

1. JavaScript运行环境，相当于js在服务端的一个宿主环境

2. 依赖于Chrome V8引擎进行代码解释，V8引擎执行Javascript的速度非常快，性能非常好。

3. 事件驱动

4. 非阻塞I/O

5. 轻量、可伸缩，适于实时数据交互应用

6. 单进程，单线程

7. Node提供核心功能模块，使得JavaScript语言可以实现文件的读写、进程管理以及网络通信等功能，在这个意义上，Node又是JavaScript的工具库。

## 为什么要学nodejs

包括但不限于以下几点。

### Nodejs市场活跃
  - 从[npmjs.org](https://www.npmjs.com/)社区可以看到，有超过20W的第三方package，每天亿级以上的下载量。
  - [github.com](https://github.com/)上搜索node相关的项目，15万个以上。
  - 而且社区非常活跃，参与的开发者众多

### **Nodejs应用广泛**
我们来看使用nodejs可以做什么：

- 项目管理：npm,grunt, gulp,bower, yeoman
- 桌面应用: node-webkit
- Web开发：express,ejs,hexo, socket.io, restify, cleaver, stylus, browserify,cheerio
- 工具包 underscore,moment,connet,later,log4js,passport,passport(oAuth),domain,require,reap,commander,retry,PDFkit
- 数据库：mysql,mongoose,redis,memcached
- 异步：async,wind,eventProxy,bluebird
- 部署：forever,pm2,nodemon
- 测试：jasmine,karma,protractor
- 跨平台：rio,tty
- 内核：cluster,http,request
- 模板: jade
- 博客: ghost,hexo
- 微信: weui
- 硬件控制: NoduinoWeb
- 操作系统: NodeOS

> 可以这么说，其他语言能做的事情，nodejs都可以做，甚至可以做的更好（不知道这样说会被会被人喷）。当然，更需要结合业务具体分析进行技术选型。

### **推荐关注**

1. [github](https://github.com/)：在这里可以找到大量nodejs相关的项目，阅读源码源码，查看新技术的一手资料

2. [nodejs官网](https://nodejs.org)：关注Node版本更新，包括api功能及使用、bug修复、新增特性以及未来的发展趋势

3. [npm官网](https://www.npmjs.com/)：在这里搜索你想用的包，参考别人的源代码

4. [stackoverflow问答社区](http://stackoverflow.com/)：有什么疑惑直接在这开问吧，会有很多热情的好基友来帮你解答问题的，比如服务异常、配置什么的。

## NodeJS优缺点及适用场景讨论

基于以上提及的Nodejs特性，我们来看在实际系统问题中是如何体现其优势的。

Nodejs的出现为我们解决现实当中系统瓶颈提供了新的思路和方案

#### 解决并发连接的问题
对于并发连接的问题讨论，为了有更直观的理解，我们来看以下三个模型：
- 系统线程模型

![](/img/model1.jpg)

这种模型的问题显而易见，服务端只有一个线程，并发请求（用户）到达只能处理一个，其余的要先等待，这就是阻塞，正在享受服务的请求阻塞后面的请求了。

- 多线程、线程池模型

![](/img/model2.jpg)

这个模型已经比上一个有所进步，它调节服务端线程的数量来提高对并发请求的接收和响应，但并发量高的时候，请求仍然需要等待。它有个更严重的问题，即服务端与客户端每建立一个连接，都要为这个连接分配一套配套的资源，主要体现为系统内存资源，以PHP为例，维护一个连接可能需要20M的内存。这就是为什么一般并发量一大，就需要多开服务器的原因。

- 异步、事件驱动模型

那么我们来看nodejs是如何处理这个问题的

![](/img/model3.jpg)

我们同样是要发起请求，等待服务器端响应；但不同的是，点完餐拿到号码后，我们往往会在位置上等待，而在我们后面的请求会继续得到处理，同样是拿了一个号码然后到一旁等待，接待员能一直进行处理。

等到饭菜做好了，会喊号码，我们拿到了自己的饭菜，进行后续的处理（吃饭）。这个喊号码的动作在NodeJS中叫做回调（Callback），能在事件（烧菜，I/O）处理完成后继续执行后面的逻辑（吃饭），这体现了NodeJS的显著特点：异步机制、事件驱动整个过程没有阻塞新用户的连接（点餐），也不需要维护已经点餐的用户与厨师的连接。

基于这样的机制，理论上陆续有用户请求连接，NodeJS都可以进行响应，因此NodeJS能支持比Java、PHP程序更高的并发量。

虽然维护事件队列也需要成本，但由于NodeJS是单线程，事件队列越长，得到响应的时间就越长，并发量上去后面对CPU密集型模型的时候就还是会力不从心，因此依然被部分人诟病。但好在，活跃的社区在持续的解决各种问题，应运而生的[threads-a-gogo（以下简称TAGG）](https://github.com/xk/node-threads-a-gogo)这个模块就是让node支持多线程模型。

> 总结一下NodeJS是怎么解决并发连接这个问题的：更改连接到服务器的方式，每个连接发射（emit）一个在NodeJS引擎进程中运行的事件（Event），放进事件队列当中，而不是为每个连接生成一个新的OS线程（并为其分配一些配套内存）。


#### 解决I/O阻塞的问题

NodeJS解决的另外一个问题是I/O阻塞，看看这样的业务场景：需要从多个数据源拉取数据，然后进行处理，处理的方式有：

1. 串行获取数据，这是我们一般的解决方案
2. NodeJS非阻塞I/O，是通过发射/监听事件来控制执行过程


NodeJS遇到I/O事件会创建一个线程去执行，然后主线程会继续往下执行，事件代理接收到线程后继续往下执行后面的逻辑，这就是NodeJS非阻塞I/O的特点。

我们来看以下示例：

```
/**
 * [require description]
 * @param  {[type]} 'events' [description]
 * @return {[type]}          [description]
 */
var EventEmitter = require('events').EventEmitter;
var event = new EventEmitter();

event.on('eat', function() {
	console.log('开饭啦');
});

console.log('我要先去散步');

setTimeout(function() {
	event.emit('eat');
}, 1000);

```

#### 综合对比分析

1. Nodejs具有处理高并发的能力（最重要的优点）
2. Nodejs适合I/O密集型应用
3. Nodejs不适合CPU密集型应用；CPU密集型应用给Node带来的挑战主要是：由于JavaScript单线程的原因，如果有长时间运行的计算（比如大循环），将会导致CPU时间片不能释放，使得后续I/O无法发起；

解决方案：分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞I/O调用的发起；或者使用第三方模块，让Node也可以创建多进程。

当然，也有很多人吐槽：

1. 只支持单核CPU，不能充分利用CPU

2. 可靠性低，一旦代码某个环节崩溃，整个系统都崩溃，原因是nodejs是单进程单线程

But，现在我们都有解决方案：
（1）Nnigx反向代理，负载均衡，开多个进程，绑定多个端口；
（2）开多个进程监听同一个端口，使用cluster模块；

戳这里了解[更多](http://taobaofed.org/blog/2015/11/03/nodejs-cluster/)

> 总而言之，NodeJS适合运用在高并发、I/O密集、少量业务逻辑的场景。


# Chapter 2 -- Node.js基础知识

> 导读：在这一部分的基础内容中，将会学习如何安装并体验Nodejs，了解Nodejs里面的模块以及相关的代码规范。同时，还可以学习npm包管理器的使用。

## 安装

### windows上安装

- 先去下载一下[git](http://git-scm.com/download/)的客户端，可以运行git bash，方便使用shell命令

`step1.` 进入[nodejs.org](https://nodejs.org/)下载

`step2.` 下载完成后，双击默认安装。安装程序会自动添加环境变量

`step3.` 检测nodejs是否安装成功。打开cmd命令行 输入 :

```
node - v
```

`step4.` 检查npm是否安装。由于新版的NodeJS已经集成了npm，所以之前npm也一并安装好了。同样可以使用cmd命令行进行确认。

```
npm -v
```

### mac上安装

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

### linux上安装

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
### 版本说明

> 目前最新的都已经到`5.1.0 `，这是从0.12版本后，nodejs和iojs(由于部分开发者对管理模式的不满，便fork了nodejs后创建了io.js，采用独立的社区驱动模式运营这个开源项目，每周一个版本迭代)合并了，直接从4.0版本开始发展了，到如今已经是5.1版本。当然，还在持续更新迭代中。

**关于nodejs版本号的说明**

偶数位的版本是稳定版本，而一般奇数位的就是非稳定版本，这几乎是在业界大家都达成共识了。比如0.6.x就是稳定版本，而0.11.x就是新功能测试的非稳定版本

建议选择最新的稳定版本进行使用。由于版本较多，为了方便node版本管理，推荐几个工具：

- osx, linux系统下

推荐使用n和nvm进行node的多版本管理，n是node的一个模块，TJ大神开发的。

```
npm install -g n
n 5.0.0
```

- windows系统下

推荐使用nvmw进行node的多版本管理。

```
// 选择一个目录下载nvmw，比如放在C:\Program Files\nvmw
git clone https://github.com/hakobera/nvmw.git
// 配置环境变量，在path中增加C:\Program Files\nvmw
// 运行nvmw命令
nvmw ls
nvmw use v5.0.0
nvmw install v5.0.0
```

不过，这个比较坑，git bash下无法运行，而且，对新版node还不支持

## node初体验

### nodejs的REPL交互环境
在浏览器中控制台，我们可以直接编写js代码进行运行

在你的CMD窗口，键入node后回车，即可进入node的repl环境运行js代码。

[更多关于REPL的简单说明戳这里](http://segmentfault.com/a/1190000002673137)

### 写个脚本

代码参见[git仓库](https://github.com/iUAP-FE/nodejs)。

### 起个web服务

```
//http.js

var http = require('http');

var app = http.createServer(function(req, res){
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello world");
});

app.listen(1337);

```

是的，就是这样神奇，短短几行代码，就创建了一个web服务，而且，请不要轻视它，这还是一个高性能的web服务器。

## Nodejs模块概述

> nodejs的模块分为提供的核心模块以及我们编写的业务模块

### 加载模块

Node.js采用模块化结构，按照CommonJS规范定义和使用模块。模块与文件是一一对应关系，即加载一个模块，实际上就是加载对应的一个模块文件。

require方法的参数是模块文件的名字。它分成两种情况。

第一种情况是参数中含有文件路径，这时路径是相对于当前脚本所在的目录：

```
var myfile = require('./myfile.js');
```

第二种情况是参数中不含有文件路径，这时Node到模块的安装目录，去寻找已安装的模块（比如下例）。

```
var gulp = require('gulp');
```

### 核心模块

如果只是在服务器运行JavaScript代码，用处并不大，因为服务器脚本语言已经有很多种了。Node.js的用处在于，它本身还提供了一系列功能模块，与操作系统互动。这些核心的功能模块，不用安装就可以使用，下面是它们的清单。

```
http：提供HTTP服务器功能。
url：解析URL。
fs：与文件系统交互。
querystring：解析URL的查询字符串。
child_process：新建子进程。
util：提供一系列实用小工具。
path：处理文件路径。
crypto：提供加密和解密功能，基本上是对OpenSSL的包装。
```

上面这些核心模块，源码都在Node的lib子目录中。为了提高运行速度，它们安装时都会被编译成二进制文件。

核心模块总是最优先加载的。如果你自己写了一个HTTP模块，`require('http')`加载的还是核心模块。


### 自定义模块

Node模块采用CommonJS规范。只要符合这个规范，就可以自定义模块。

下面是一个最简单的模块，假定新建一个test.js文件，写入以下内容。

```
module.exports = function(str) {
    alert(str);
};
```

上面代码就是一个模块，它通过module.exports变量，对外输出一个方法。

这个模块的使用方法如下。

```
var test = require('./test');

test("hello world");
```

上面代码通过require命令加载模块文件test.js（后缀名省略）。


## commonjs代码规范说明

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

### module对象
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
### require命令的解读
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

### require加载模块的规则
require命令用于加载文件，可以加载后缀名为`.js` `.json` `.node`的文件，比如：
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

require匹配文件的流程图例

![](/img/require.jpg)


## npm包管理器

### 介绍

npm有两层含义：
* Node.js的开放式模块登记和管理系统，网址为[http://npmjs.org](http://npmjs.org)。
* Node.js默认的模块管理器，是一个命令行下的软件，用来安装和管理node模块。

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：

允许用户从NPM服务器下载别人编写的第三方包到本地使用。
允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

### 全局配置
`step1.` npm作为一个NodeJS的模块管理，我们要先配置npm的全局模块的存放路径以及cache的路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立**node_global**及**node_cache**两个文件夹。我们就在cmd中键入两行命令：
```
npm config set prefix "D:\Program Files\nodejs\node_global"
npm config set cache "D:\Program Files\nodejs\node_cache"
```
`step2.` 下面这一步非常关键，我们需要设置系统变量。进入我的电脑→属性→高级→环境变量。
```
在系统变量下新建“NODE_PATH”
输入内容“D:\Program Files\nodejs\node_global\node_modules”
```
`step3.` 安装bower或是gulp等常用工具
```
npm install bower gulp -g
```
这个时候，你在命令行就可以全局的使用安装的工具了，可以命令行中直接确认
```
gulp -v
bower -v
```
`step4.` 接下来我们可以进一步学习如何用npm发布自己的包
```
// 初始化一个package
npm init
// 添加你的包的管理用户
npm adduser
// 发布你的包
npm publish --tag 0.1.0
```

### 学习使用

1. npm set
npm set用来设置环境变量。

2. npm info
npm info命令可以查看每个模块的具体信息

3. npm search
npm search命令用于搜索npm仓库，它后面可以跟字符串，也可以跟正则表达式。

4. npm install
Node模块采用npm install命令安装。每个模块可以“全局安装”，也可以“本地安装”。两者的差异是模块的安装位置，以及调用方法。

5. npm update
npm update 命令可以升级本地安装的模块。

6. npm uninstall
卸载掉安装的模块

7. npm publish
发布你的包
8. npm adduser
给你的包添加用户信息
9. npm cache clear
清除缓存

# Chapter 3 -- Nodejs中的对象和核心API

## node中常用的全局对象

> 浏览器中运行的js可以访问全局对象window，同样的，在node中运行的js可以访问全局对象global，下面一起了解下在nodejs中都有哪些非常有用的全局对象。


`global`是全局命名空间对象。

这里需要说明的是，在浏览器中，顶级作用域就是全局作用域。这就是说，在浏览器中，如果当前是在全局作用域内，var something将会声明一个全局变量。在Node中则不同。顶级作用域并非全局作用域，在Node模块里的var something只属于那个模块。

### Console对象

console 用于提供控制台标准输出，它是由 Internet Explorer 的 JScript 引擎提供的调试工具，后来逐渐成为浏览器的事实标准。 Node.js 沿用了这个标准，提供与习惯行为一致的 console 对象，用于向标准输出流（stdout）或标准错误流（stderr）输出字符。

console 方法 以下为 console 对象的方法:

- console.log([data][, ...]) 向标准输出流打印字符并以换行符结束。该方法接收若干 个参数，如果只有一个参数，则输出这个参数的字符串形式。如果有多个参数，则 以类似于C 语言 printf() 命令的格式输出。
- console.info([data][, ...]) P该命令的作用是返回信息性消息，这个命令与console.log差别并不大，除了在chrome中只会输出文字外，其余的会显示一个蓝色的惊叹号。
- console.error([data][, ...]) 输出错误消息的。控制台在出现错误时会显示是红色的叉子。
- console.warn([data][, ...]) 输出警告消息。控制台出现有黄色的惊叹号。
- console.dir(obj[, options]) 用来对一个对象进行检查（inspect），并以易于阅读和打印的格式显示。
- console.time(label) 输出时间，表示计时开始。
- console.timeEnd(label) 结束时间，表示计时结束。
- console.trace(message[, ...]) 当前执行的代码在堆栈中的调用路径，这个测试函数运行很有帮助，只要给想测试的函数里面加入 - console.trace 就行了。
- console.assert(value[, message][, ...]) 用于判断某个表达式或变量是否为真，接手两个参数，第一个参数是表达式，第二个参数是字符串。只有当第一个参数为false，才会输出第二个参数，否则不会有任何结果。
- console.log()：向标准输出流打印字符并以换行符结束。 console.log 接受若干 个参数，如果只有一个参数，则输出这个参数的字符串形式。如果有多个参数，则 以类似于C 语言 printf() 命令的格式输出。 第一个参数是一个字符串，如果没有 参数，只打印一个换行。

```
console.log('Hello world');
console.log('byvoid%diovyb');
console.log('byvoid%diovyb', 1991);
console.error()：与console.log() 用法相同，只是向标准错误流输出。 console.trace()：向标准错误流输出当前的调用栈。
```
### Buffer对象

Buffer对象Node原生提供的全局对象，用来处理二进制数据的一个接口。JavaScript比较擅长处理Unicode数据，对于处理二进制格式的数据（比如TCP数据流），就不太擅长。Buffer对象就是为了解决这个问题而提供的。该对象也是一个构造函数，它的实例代表了V8引擎分配的一段内存，基本上是一个数组，成员都为整数值。

> API说明

```javascript
{ [Function: Buffer]
  poolSize: 8192,
  isBuffer: [Function: isBuffer],
  compare: [Function: compare],
  isEncoding: [Function],
  concat: [Function],
  byteLength: [Function: byteLength] }
```

暂时存放的一块内存，处理二进制类型文件
创建buffer有三种办法：
- `new Buffer(size)`指定长度，然后fill填充内容
- `new Buffer(['sss', 'xxx'])`传入一个数组
- `new Buffer('郭永峰')`传入一个字符串

### 几个重要的模块内部的局部变量

模块内部的局部变量，指向的对象根据模块不同而不同，但是所有模块都适用，可以看作是伪全局变量， 主要为__filename,__dirname,module, module.exports, exports等。

**__filename**
注意此属性并不是全局对象的属性，而只是node在我们注入模块的参数，可以在模块内直接使用. _filename 表示当前正在执行的脚本的文件名。它将输出文件所在位置的绝对路径

 // 输出全局变量 __filename 的值
 console.log( __filename );
**__dirname**
注意此属性并不是全局对象的属性，而只是node为我们注入模块的参数，可以在模块内直接使用. __dirname 表示当前执行脚本所在的目录。

// 输出全局变量 __dirname 的值
console.log( __dirname );
**module**
代表当前模块本身

**exports**
模块的导出对象


### path对象

path.join方法用于连接路径。该方法的主要用途在于，会正确使用当前系统的路径分隔符，Unix系统是”/“，Windows系统是”\“。

API接口有：`normalize` `join` `resolve` `parse` `dirname` `basename` `

**path.resolve()**：一个重要的方法

path.resolve方法用于将相对路径转为绝对路径。

它可以接受多个参数，依次表示所要进入的路径，直到将最后一个参数转为绝对路径。如果根据参数无法得到绝对路径，就以当前所在路径作为基准。除了根目录，该方法的返回值都不带尾部的斜杠。

```
// 格式
path.resolve([from ...], to)

// 实例
path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
```
上面代码的实例，执行效果类似下面的命令。

bash

$ cd foo/bar
$ cd /tmp/file/
$ cd ..
$ cd a/../subfile
$ pwd

```
path.resolve('/foo/bar', './baz')
// '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/')
// '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
// 如果当前目录是/home/myself/node，返回
// /home/myself/node/wwwroot/static_files/gif/image.gif
该方法忽略非字符串的参数。
```

### stream流对象

Stream把较大的数据，拆成很小的部分。只要命令部署了Stream接口，就可以把一个流的输出接到另一个流的输入。Node引入了这个概念，通过Stream为异步读写数据提供的统一接口。无论是硬盘数据、网络数据，还是内存数据，都可以采用这个接口读写。

后面接触的文件系统fs就是基于stream来实现，比较常用的构建工具gulp也是基于流来工作的。

一个典型的写文件操作：

```
var http = require('http');
var fs = require('fs');

var server = http.createServer(function (req, res) {
  fs.readFile(__dirname + '/data.txt', function (err, data) {
    res.end(data);
  });
});

server.listen(8000);
```

Stream接口分成三类。

- 可读数据流接口，用于读取数据。

```
var Readable = require('stream').Readable;

var rs = new Readable; rs.push('beep '); rs.push('boop\n'); rs.push(null);

rs.pipe(process.stdout);
```


- 可写数据流接口，用于写入数据。

```
var fs = require('fs'); var readableStream = fs.createReadStream('file1.txt'); var writableStream = fs.createWriteStream('file2.txt');

readableStream.setEncoding('utf8');

readableStream.on('data', function(chunk) { writableStream.write(chunk); });
```

- 双向数据流接口，用于读取和写入数据，比如Node的tcp、sockets、zlib、crypto都部署了这个接口。

### process

API接口：argv pid kill stdout stderr strin console nextTick 等

process 是一个全局变量，即 global 对象的属性。 它用于描述当前Node.js 进程状态的对象，提供了一个与操作系统的简单接口。通常在你写本地命令行程序的时候，少不了要 和它打交道。下面将会介绍 process 对象的一些最常用的成员方法。

**process.arg**
是命令行参数数组，第一个元素是node.exe可执行文件的路径,第二个参数是执行的脚本文件所有路径，从第三个元素开始每个参数是一个运行时传入的参数

console.log(process.argv);
以将上代码保存为argv.js,然后运行

```
E:\nodejs\test>node argv.js test input
```

得到结果

```
[ 'C:\\Program Files\\nodejs\\node.exe',
  'E:\\testjus\\argv.js',
  'test',
  'input' ]
```

**process.stdout**
标准输出流，通常我们使用的console.log往标准输出打印字符，其实调用的就是process.stdout.write()函数

**process.stdin**
标准输入流，初始时它是暂停的，要想从标准输入读取数据，必须恢复流，并手动编写流的事件响应函数。

```
process.stdin.resume();

process.stdin.on('data',function(data){
  process.stdout.write('从控制台读入用户输入的字符'+data.toString());
});
```

## 学习node中的核心模块

### Fs模块

fs是filesystem的缩写，该模块提供本地文件的读写能力，基本上是POSIX文件操作命令的简单包装。但是，这个模块几乎对所有操作提供异步和同步两种操作方式，供开发者选择。

readFileSync()
readFileSync方法用于同步读取文件，返回一个字符串。

```
var text = fs.readFileSync(fileName, "utf8");

// 将文件按行拆成数组
text.split(/\r?\n/).forEach(function (line) {
  // ...
});
```

writeFileSync()
writeFileSync方法用于同步写入文件。

```
fs.writeFileSync(fileName, str, 'utf8');
```

mkdir()，writeFile()，readfile()
mkdir方法用于新建目录。

```
var fs = require('fs');

fs.mkdir('./helloDir',0777, function (err) {
  if (err) throw err;
});
```

mkdir接受三个参数，第一个是目录名，第二个是权限值，第三个是回调函数。

writeFile方法用于写入文件。

```
var fs = require('fs');

fs.writeFile('./helloDir/message.txt', 'Hello Node', function (err) {
  if (err) throw err;
  console.log('文件写入成功');
});
```

readfile方法用于读取文件内容。

```
var fs = require('fs');

fs.readFile('./helloDir/message.txt','UTF-8' ,function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

上面代码使用readFile方法读取文件。readFile方法的第一个参数是文件名，第二个参数是文件编码，第三个参数是回调函数。可用的文件编码包括“ascii”、“utf8”和“base64”。如果没有指定文件编码，返回的是原始的缓存二进制数据，这时需要调用buffer对象的toString方法，将其转为字符串。

```
var fs = require('fs');
fs.readFile('example_log.txt', function (err, logData) {
  if (err) throw err;
  var text = logData.toString();
});
```

readFile方法是异步操作，所以必须小心，不要同时发起多个readFile请求。

```
for(var i = 1; i <= 1000; i++) {
  fs.readFile('./'+i+'.txt', function() {
     // do something with the file
  });
}
```

上面代码会同时发起1000个readFile异步请求，很快就会耗尽系统资源。

### Http模块

Http模块主要用于搭建HTTP服务。使用Node.js搭建HTTP服务器非常简单。

* 处理GET请求

```
var http = require("http");

http.createServer(function(req, res) {

// 主页
if (req.url == "/") {
  res.writeHead(200, { "Content-Type": "text/html" });
  res.end("Welcome to the homepage!");
}

// About页面
else if (req.url == "/about") {
  res.writeHead(200, { "Content-Type": "text/html" });
  res.end("Welcome to the about page!");
}

// 404错误
else {
  res.writeHead(404, { "Content-Type": "text/plain" });
  res.end("404 error! File not found.");
}
}).listen(8080, "localhost");
```

上面代码第一行var http = require("http")，表示加载http模块。然后，调用http模块的createServer方法，创造一个服务器实例，将它赋给变量http。

ceateServer方法接受一个函数作为参数，该函数的request参数是一个对象，表示客户端的HTTP请求；response参数也是一个对象，表示服务器端的HTTP回应。response.writeHead方法表示，服务器端回应一个HTTP头信息；response.end方法表示，服务器端回应的具体内容，以及回应完成后关闭本次对话。最后的listen(8080)表示启动服务器实例，监听本机的8080端口。

将上面这几行代码保存成文件app.js，然后用node调用这个文件，服务器就开始运行了。

* 处理POST请求

当客户端采用POST方法发送数据时，服务器端可以对data和end两个事件，设立监听函数。

```
var http = require('http');

http.createServer(function (req, res) { var content = "";

req.on('data', function (chunk) {
  content += chunk;
});

req.on('end', function () {
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.write("You've sent: " + content);
  res.end();
});
}).listen(8080);
```

**模块属性**

（1）HTTP请求的属性

```
headers：HTTP请求的头信息。
url：请求的路径。
```
**模块方法**

（1）http模块的方法

```
createServer(callback)：创造服务器实例。
```

（2）服务器实例的方法

```
listen(port)：启动服务器监听指定端口。
```

（3）HTTP回应的方法

```
setHeader(key, value)：指定HTTP头信息。
write(str)：指定HTTP回应的内容。
end()：发送HTTP回应。
```

### Events模块和异步编程

Events模块是node.js对“发布/订阅”模式（publish/subscribe）的部署。一个对象通过这个模块，向另一个对象传递消息。该模块通过EventEmitter属性，提供了一个构造函数。该构造函数的实例具有on方法，可以用来监听指定事件，并触发回调函数。任意对象都可以发布指定事件，被EventEmitter实例的on方法监听到。

下面是一个实例，先建立一个消息中心，然后通过on方法，为各种事件指定回调函数，从而将程序转为事件驱动型，各个模块之间通过事件联系。

```
var EventEmitter = require("events").EventEmitter;

var ee = new EventEmitter();
ee.on("someEvent", function () {
  console.log("event has occured");
});

ee.emit("someEvent");
```

Events模块的作用，还在于其他模块可以部署EventEmitter接口，从而也能够订阅和发布消息。

```
var EventEmitter = require('events').EventEmitter;

function Dog(name) {
  this.name = name;
}

Dog.prototype.__proto__ = EventEmitter.prototype;
// 另一种写法
// Dog.prototype = Object.create(EventEmitter.prototype);

var simon = new Dog('simon');

simon.on('bark', function(){
  console.log(this.name + ' barked');
});

setInterval(function(){
  simon.emit('bark');
}, 500);

```

上面代码新建了一个构造函数Dog，然后让其继承EventEmitter，因此Dog就拥有了EventEmitter的接口。最后，为Dog的实例指定bark事件的监听函数，再使用EventEmitter的emit方法，触发bark事件。

# Chapter 4 -- Node.js开发实战

> 导读：通过对express、jade、angular、mongodb等技术结合的一个示例代码，和大家分享涉及的技术点。

## express的使用

### 介绍
基于 Node.js 平台，快速、开放、极简的 web 开发框架。

### 安装

```
npm install express express-generator -g
```

### 生成项目脚手架代码

```
express myapp
```

## ejs

jade本身是基于nodejs开发的模板引擎

## angular

angular是一个前端MVC框架，在这里会和大家简单介绍基于nodejs的项目对angular的使用

## mongodb

mongodb是目前流行的noSQL数据库，因其简单实用而流行。
