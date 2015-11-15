<h1 style="text-align:center;color:blue;font-size:44px;">Node.js知识学习</h1>

<img width="840" src="/img/node.png" />

# Chapter 1 -- Node.js入门知识

> 导读：在这一部分的内容里，主要会简单介绍一下什么是Node.js以及为什么我们要去学习它，Nodej有哪些用武之地，我们为什么要去学习它。另外，还会和大家聊一下简单的安装过程以及跟Nodejs版本相关的故事。

## 基本介绍

**Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 的包管理器 npm，是全球最大的开源库生态系统。**

这是最官方的Nodejs介绍。

对于以上提到的**运行环境**，我们可以这样理解：
> JS本身是脚本语言，脚本语言都需要一个解析器才能运行。对于写在HTML页面里的JS，浏览器充当了解析器的角色。
> 而对于需要独立运行的JS，NodeJS就是一个解析器，是JS语言的服务器运行环境，可以使JS流畅的运行于服务器端。

1. 首先要避免一个误区，如今有无数的前端框架和类库都是各种`.js`结尾，虽然Node.js也是这样，但请不要混淆概念。
2. JavaScript就好比是找到了一个宿主环境，可以通过Node在服务器端运行。
3. Node提供大量工具模块、丰富的第三方包以及基于node的web框架，使得JavaScript语言可以实现文件的读写、进程管理以及网络通信等，在这个意义上，Node又是JavaScript的工具库。
4. 而且，Node内部采用Google公司的V8引擎，作为JavaScript语言解释器；通过自行开发的libuv库，调用操作系统资源。

## 为什么要学nodejs

### **nodejs很火**
  - [npmjs.org](https://www.npmjs.com/)社区的活跃程度可以直接访问网站即可看到，大量的第三方包，伸手党的天堂。
  - [github.com](https://github.com/)上搜索node相关的项目，15万个以上。

### **nodejs很强**
我们来看使用nodejs可以做什么（整理自张仁阳老师）：

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

> 其他语言能做的事情，nodejs都可以做，甚至可以做的更好。当然，更需要结合业务具体分析进行技术选型。

### **生态完善**
  - 社区欣欣向荣
  - 丰富的第三方包

### **推荐关注**

1. [github](https://github.com/)在这里可以找到大量nodejs相关的项目，阅读源码源码，查看新技术的一手资料

2. [nodejs官网](https://nodejs.org)关注Node版本更新，包括api功能及使用、bug修复、新增特性以及未来的发展趋势

3. [npm官网](https://www.npmjs.com/)在这里搜索你想用的包，参考别人的源代码

4. [stackoverflow](http://stackoverflow.com/)问答社区，有什么疑惑直接在这开问吧，会有很多热情的好基友来帮你解答问题的，比如服务异常、配置什么的。

## NodeJS优缺点及适用场景讨论

### 特点
1. 它是一个Javascript运行环境

2. 依赖于Chrome V8引擎进行代码解释

3. 事件驱动

4. 非阻塞I/O

5. 轻量、可伸缩，适于实时数据交互应用

6. 单进程，单线程

下面我们看以上特性在实际系统问题中是如何体现其优势的。

### Nodejs带来的对系统瓶颈的解决方案
> Nodejs的出现为我们解决现实当中系统瓶颈提供了新的思路和方案

#### 解决并发连接的问题
在并发连接的问题讨论上，问了有更直观的理解，下面我们来看以下三个模型：
- 系统线程模型

![](/img/model1.jpg)

这种模型的问题显而易见，服务端只有一个线程，并发请求（用户）到达只能处理一个，其余的要先等待，这就是阻塞，正在享受服务的请求阻塞后面的请求了。

- 多线程、线程池模型
![](/img/model2.jpg)
这个模型已经比上一个有所进步，它调节服务端线程的数量来提高对并发请求的接收和响应，但并发量高的时候，请求仍然需要等待。它有个更严重的问题，即服务端与客户端每建立一个连接，都要为这个连接分配一套配套的资源，主要体现为系统内存资源，以PHP为例，维护一个连接可能需要20M的内存。这就是为什么一般并发量一大，就需要多开服务器。

- 异步、事件驱动模型
那么我们来看nodejs是如何处理这个问题的

![](/img/model3.jpg)
我们同样是要发起请求，等待服务器端响应；但是与银行例子不同的是，这次我们点完餐后拿到了一个号码，拿到号码，我们往往会在位置上等待，而在我们后面的请求会继续得到处理，同样是拿了一个号码然后到一旁等待，接待员能一直进行处理。

等到饭菜做号了，会喊号码，我们拿到了自己的饭菜，进行后续的处理（吃饭）。这个喊号码的动作在NodeJS中叫做回调（Callback），能在事件（烧菜，I/O）处理完成后继续执行后面的逻辑（吃饭），这体现了NodeJS的显著特点，异步机制、事件驱动整个过程没有阻塞新用户的连接（点餐），也不需要维护已经点餐的用户与厨师的连接。

基于这样的机制，理论上陆续有用户请求连接，NodeJS都可以进行响应，因此NodeJS能支持比Java、PHP程序更高的并发量虽然维护事件队列也需要成本，再由于NodeJS是单线程，事件队列越长，得到响应的时间就越长，并发量上去还是会力不从心。

> 总结一下NodeJS是怎么解决并发连接这个问题的：更改连接到服务器的方式，每个连接发射（emit）一个在NodeJS引擎进程中运行的事件（Event），放进事件队列当中，而不是为每个连接生成一个新的OS线程（并为其分配一些配套内存）。


#### 解决I/O阻塞

NodeJS解决的另外一个问题是I/O阻塞，看看这样的业务场景：需要从多个数据源拉取数据，然后进行处理。

- 串行获取数据，这是我们一般的解决方案
- NodeJS非阻塞I/O，发射/监听事件来控制执行过程

NodeJS遇到I/O事件会创建一个线程去执行，然后主线程会继续往下执行的，因此，拿profile的动作触发一个I/O事件，马上就会执行拿timeline的动作，两个动作并行执行，假如各需要1S，那么总的时间也就是1S。它们的I/O操作执行完成后，发射一个事件，profile和timeline，事件代理接收后继续往下执行后面的逻辑，这就是NodeJS非阻塞I/O的特点。

> 总结一下：Java、PHP也有办法实现并行请求（子线程），但NodeJS通过回调函数（Callback）和异步机制会做得很自然。

#### 综合对比分析

1. Nodejs具有处理高并发的能力（最重要的优点）
2. Nodejs适合I/O密集型应用
3. Nodejs不适合CPU密集型应用；CPU密集型应用给Node带来的挑战主要是：由于JavaScript单线程的原因，如果有长时间运行的计算（比如大循环），将会导致CPU时间片不能释放，使得后续I/O无法发起；

解决方案：分解大型运算任务为多个小任务，使得运算能够适时释放，不阻塞I/O调用的发起；

4. 只支持单核CPU，不能充分利用CPU

5. 可靠性低，一旦代码某个环节崩溃，整个系统都崩溃，原因是nodejs是单进程单线程

解决方案：
（1）Nnigx反向代理，负载均衡，开多个进程，绑定多个端口；
（2）开多个进程监听同一个端口，使用cluster模块；


> 总而言之，NodeJS适合运用在高并发、I/O密集、少量业务逻辑的场景。其实NodeJS能实现几乎一切的应用，我们考虑的点只是适不适合用它来做。


# Chapter 2 -- Node.js基础知识
> 导读：在基础部分里面，我们会有很多的内容学习。
- node初体验
- commmonjs模块说明
- nodejs全局对象的学习
- nodejs的核心模块的学习


## 安装

### windows上安装

- 先去下载一下git的客户端，可以运行git bash，方便使用shell命令

`step1.` 进入nodejs官方网站下载软件(nodejs.org),
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

> 目前最新的都已经到`5.0.0`，这是从0.12版本后，nodejs和iojs合并了，直接从4.0版本开始发展了，到如今已经是5.0版本，当然，还在持续更新迭代中。

版本不同，接口API都是不一样的 要关注

关于nodejs版本号的说明

偶尔位的版本是稳定版本，而一般奇数位的就是非稳定版本，这几乎是在业界大家都达成共识了。比如0.6.x就是稳定版本，而0.11.x就是新功能测试的非稳定版本

建议选择最新的稳定版本进行使用

而对于node版本管理的问题，推荐几个工具：
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

### 起个web服务器

//server.js
```
var http = require('http')
http.createServer(function(req, res){
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello world");
  }).listen(1337, "127.0.0.1");

```

是的，就是这样神奇，短短几行代码，就创建了一个web服务，而且，请不要轻视它，这还是一个高性能的web服务器。

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

## 模块和包

基于以上的学习，我们大体理解了node是什么，而且会基本的node程序的书写，现在来进一步理解模块和包，进入学习node的快车道。

### 概念区分

在node里面，单个的文件即模块，每个模块可以提供很多个API接口进行使用。

而包的概念，比模块大，多个模块可以封装成一个包（甚至一个也行），且包的封装和发布，使用和更新，都需要遵循包的相关机制。包的管理可以使用npm，npm是最大的包管理和分发平台。

这里提到的npm有两层含义：
1. Node.js的开放式模块登记和管理系统。
2. Node.js默认的模块管理器，是一个命令行下的软件，用来安装和管理node模块。

### 模块分类
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

### 包的概念和npm的配置

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

### 学习封装一个包

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

## node中常用的全局对象
> 浏览器中运行的js可以访问全局对象window，同样的，在node中运行的js可以访问全局对象global，下面一起了解下在nodejs中都有哪些非常有用的全局对象。

### global

`global`是全局命名空间对象。
这里需要说明的是，在浏览器中，顶级作用域就是全局作用域。这就是说，在浏览器中，如果当前是在全局作用域内，var something将会声明一个全局变量。在Node中则不同。顶级作用域并非全局作用域，在Node模块里的var something只属于那个模块。

### Buffer对象

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

### path对象

> API接口说明

```
{ resolve: [Function],
  normalize: [Function],
  isAbsolute: [Function],
  join: [Function],
  relative: [Function],
  _makeLong: [Function],
  dirname: [Function],
  basename: [Function],
  extname: [Function],
  format: [Function],
  parse: [Function],
  sep: '\\',
  delimiter: ';',
  posix:
   { resolve: [Function],
     normalize: [Function],
     isAbsolute: [Function],
     join: [Function],
     relative: [Function],
     _makeLong: [Function],
     dirname: [Function],
     basename: [Function],
     extname: [Function],
     format: [Function],
     parse: [Function],
     sep: '/',
     delimiter: ':' },
  win32: [Circular] }
```

path用于处理路径相关

API接口：`normalize` `join` `resolve` `parse` `dirname` `basename` `

### stream流对象

在网络中传输数据的时候，总是先将对象转化为流数据，也就是字节数组，再通过流的传输，到达目的地后再转成(内容类型和编码)原始的数据
- `stream.Readable`
- fs文件系统对象的有`fs.createReadStream`

### process
API接口：argv pid kill stdout stderr strin console nextTick 等

### require

### module

### __filename和__dirname

### console
- console.log
- console.info
- console.error
- console.timeEnd
- console.assert

## 学习node中的核心模块

### Url模块

url和uri的区别
url是uri的一个子集
url有自己的属性规则，编码

```
parse
format
resolve
```

### Path模块

字符串解析

### Fs模块

> 示例：遍历目录，操作文件，以前抓取数据的示例

### Http模块

http原理说明：
- 什么是http协议
	- 交互
	- 响应
- 普通网站的访问过程来解析http协议
- node里面的http模块的使用


### Cluster模块

- process
- cluster

### Events模块和异步编程

- promise
- 异常处理
- 处理回调深渊

# Chapter 3 -- Nodejs进阶知识
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

# Chapter 4 -- Node.js开发实战
> 导读：本章内容将会和大家分享项目实战的一些经验和遇到的问题，内容待补充，敬请期待...
