# NodeJS学习

![node](/img/node.png)

## 0 介绍

Nodejs的出现让js可以流畅的运行于服务器端，打破了前后端的局限，解放了技术人员合作的模式。让一个全新的开发模式--全栈开发，成为可能，让web开发走向了一个新的舞台。

本质上是一个js的运行环境，虽然带着.js的后缀，基于google chrome的v8引擎而且还使用了部分c++的模块。

提供许多的系统级别的API，还可以解析js代码，还没有浏览器的安全级别的限制
- 文件的读写--fs模块
- 进程的管理--cluster process child_process
- 网络的通信--http模块
等

特性
- es6的支持
- 有什么特性
- 适合的业务场景
- 全栈开发
- v8引擎

## 1 为什么要学nodejs

- 它很火，npmjs。org社区的活跃程度，大量的包，伸手党的天堂，数字说明
- 社区欣欣向荣
- github，node相关的项目 10万个
- 很强，使用node做的产品：node-webkit appjs jade grunt express nodejsos ghost log.io pdfkit

其他语言能做的事情，nodejs都可以做，甚至可以做的更好

而且，而且，对于一个前端工程师而言，我实在是找不出一个不学nodejs的理由

**推荐关注**
- github 大量nodejs相关的项目 源码 阅读
- nodejs.org 版本更新 api bug修复 新特性 未来的发展趋势
- npmjs.org 搜索包 参考别人的源代码
- stackoverflow 问答 服务异常 配置 好基友来帮你解答

**建议**
- 一定的js基础

## 2 安装

### 2.1 windows上安装

- 先去下载一下git的客户端，可以运行git bash。在这里可以运行shell命令

1. 进入nodejs官方网站下载软件(nodejs.org),
2. 下载完成后，双击默认安装。安装程序会自动添加环境变量
3. 检测nodejs是否安装成功。打开cmd命令行 输入 :
```
node - v
```
4. 检查npm是否安装。由于新版的NodeJS已经集成了npm，所以之前npm也一并安装好了。同样可以使用cmd命令行进行确认。
```
npm -v
```
5. npm作为一个NodeJS的模块管理，我们要先配置npm的全局模块的存放路径以及cache的路径，例如我希望将以上两个文件夹放在NodeJS的主目录下，便在NodeJs下建立**node_global**及**node_cache**两个文件夹。我们就在cmd中键入两行命令：
```
npm config set prefix "D:\Program Files\nodejs\node_global"
npm config set cache "D:\Program Files\nodejs\node_cache"
```
6. 下面这一步非常关键，我们需要设置系统变量。进入我的电脑→属性→高级→环境变量。
```
在系统变量下新建“NODE_PATH”
输入内容“D:\Program Files\nodejs\node_global\node_modules”
```
7. 安装bower或是gulp等常用工具
```
npm install bower gulp -g
```
这个时候，你在命令行就可以全局的使用安装的工具了，可以命令行中直接确认
```
gulp -v
bower -v
```
8. 接下来我们可以进一步学习如何用npm发布自己的包
```
// 初始化一个package
npm init
// 添加你的包的管理用户
npm adduser
// 发布你的包
npm publish --tag 0.1.0
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

> 目前最新的都已经到`5.0.0`

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


### 3.2 起个web服务器
//server.js

```
var http = require('http')
http.createServer(function(req, res){
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello world");
  }).listen(1337, "127.0.0.1");

// 链式改成回调

```

修改后需要重启一次服务

### 3.3 在命令行中直接感受
//浏览器中js环境，控制台的console
//node的repl环境
演示

说明宿主环境的全局变量

window和document
process

### 3.1 commonjs代码规范说明
每个模块都是独立
模块管理机制
- require引用
- exports导出
- module定义

每个js文件都是一个独立的模块
整个文件的代码都是独立的，也可以相互引用

## 4. 模块和包

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
### 4.1 概念区分

包和模块是不一样的，概念区分，并进行学习

### 4.2 npm

最大的包分发平台，**重要性说明**

地址：`npmjs.org`
- 如何设置
```
npm config相关
```
- 封装包
- 下载第三方包
- 发布
- 版本号

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

## 5 学习一些API

关注nodejs的版本特性，关注io.js
可以去官网看一下nodejs的接口文档，也是可以直接学习的

### url模块

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

### querystring

字符串解析

### 5.1 学习文件操作

- buffer
- stream
- fs
- path
- stdio stdout

> 示例：遍历目录，操作文件，以前抓取数据的示例

### 5.2 学习网络相关操作

- http：起个服务
- https
- url设计
- query string

> 示例：

### 5.3 学习进程的管理

- process
- cluster

### 5.4 异步编程

- promise
- 异常处理
- 处理回调深渊

## 6 使用express做个示例

### 6.1 express简介

基于 Node.js 平台，快速、开放、极简的 web 开发框架。

### 6.2 安装

```
npm install express express-generator -g
```

### 6.3 模板jade

### 6.4 写示例代码

### 6.5 优化
