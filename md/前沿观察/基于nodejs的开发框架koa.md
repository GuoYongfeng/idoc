# 基于Nodejs.js平台的下一代web开发框架koa

## 简介
koa 是由 Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。使用 koa 编写 web 应用，通过组合不同的 generator，可以免除重复繁琐的回调函数嵌套，并极大地提升错误处理的效率。koa 不在内核方法中绑定任何中间件，它仅仅提供了一个轻量优雅的函数库，使得编写 Web 应用变得得心应手。

## 安装

> Koa 目前需要 >=0.11.x版本的 node 环境。

在你的项目中安装koa
```
npm install koa --save
```

需要在执行 node 的时候附带 --harmony 来引入 generators 。如：
```
node --harmony app.js
```

## 使用koa

Koa 应用是一个包含一系列中间件 generator 函数的对象。 这些中间件函数基于 request 请求以一个类似于栈的结构组成并依次执行。 Koa 类似于其他中间件系统（比如 Ruby's Rack 、Connect 等）， 然而 Koa 的核心设计思路是为中间件层提供高级语法糖封装，以增强其互用性和健壮性，并使得编写中间件变得相当有趣。

Koa 包含了像 content-negotiation（内容协商）、cache freshness（缓存刷新）、proxy support（代理支持）和 redirection（重定向）等常用任务方法。 与提供庞大的函数支持不同，Koa只包含很小的一部分，因为Koa并不绑定任何中间件。

### 用koa写一个hello world

```
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

### 中间件的级联

Koa 的中间件通过一种更加传统（您也许会很熟悉）的方式进行级联，摒弃了以往 node 频繁的回调函数造成的复杂代码逻辑。 我们通过 generators 来实现“真正”的中间件。 Connect 简单地将控制权交给一系列函数来处理，直到函数返回。 与之不同，当执行到 yield next 语句时，Koa 暂停了该中间件，继续执行下一个符合请求的中间件('downstrem')，然后控制权再逐级返回给上层中间件('upstream')。

**示例：**

```
var koa = require('koa');
var app = koa();

// x-response-time

app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  this.set('X-Response-Time', ms + 'ms');
});

// logger

app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  console.log('%s %s - %s', this.method, this.url, ms);
});

// response

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

**代码分析：**

当请求开始时，请求先经过 x-response-time 和 logging 中间件，并记录中间件执行起始时间。 然后将控制权交给 reponse 中间件。当中间件运行到 yield next 时，函数挂起并将控制前交给下一个中间件。当没有中间件执行 yield next 时，程序栈会逆序唤起被挂起的中间件来执行接下来的代码。

### koa的简单配置

应用配置是 app 实例属性，目前支持的配置项如下：

- app.name 应用名称（可选项）
- app.env 默认为 NODE_ENV 或者 development
- app.proxy 如果为 true，则解析 "Host" 的 header 域，并支持 X-Forwarded-Host
- app.subdomainOffset 默认为2，表示 .subdomains 所忽略的字符偏移量。

## 带你了解koa的API

### app.listen(...)

Koa 应用并非是一个 1-to-1 表征关系的 HTTP 服务器。 一个或多个Koa应用可以被挂载到一起组成一个包含单一 HTTP 服务器的大型应用群。

如下为一个绑定3000端口的简单 Koa 应用，其创建并返回了一个 HTTP 服务器，为 Server#listen() 传递指定参数

示例代码：
```
var koa = require('koa');
var app = koa();
app.listen(3000);
```

这么轻轻松松就创建了一个http服务，简直是碉堡了，其实简洁的代码是由于koa对listen方法进行封装，实现的过程是这样的。

```
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
```

这意味着您可以同时支持 HTTPS 和 HTTPS，或者在多个端口监听同一个应用。
```
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
http.createServer(app.callback()).listen(3001);
```

### app.callback()

返回一个适合 http.createServer() 方法的回调函数用来处理请求。 您也可以使用这个回调函数将您的app挂载在 Connect/Express 应用上。

### 更多API

详情请看这里：`https://github.com/koajs/koa/wiki`。直接上koa的github wiki看。

### 错误处理

默认情况下Koa会将所有错误信息输出到 stderr，除非 NODE_ENV 是 "test"。为了实现自定义错误处理逻辑（比如 centralized logging），您可以添加 "error" 事件监听器。
```
app.on('error', function(err){
  log.error('server error', err);
});
```
如果错误发生在 请求/响应 环节，并且其不能够响应客户端时，Contenxt 实例也会被传递到 error 事件监听器的回调函数里。
```
app.on('error', function(err, ctx){
  log.error('server error', err, ctx);
});
```
当发生错误但仍能够响应客户端时（比如没有数据写到socket中），Koa会返回一个500错误(Internal Server Error)。

无论哪种情况，Koa都会生成一个应用级别的错误信息，以便实现日志记录等目的。

## Context

Koa Context 将 node 的 request 和 response 对象封装在一个单独的对象里面，其为编写 web 应用和 API 提供了很多有用的方法。

这些操作在 HTTP 服务器开发中经常使用，因此其被添加在上下文这一层，而不是更高层框架中，因此将迫使中间件需要重新实现这些常用方法。

context 在每个 request 请求中被创建，在中间件中作为接收器(receiver)来引用，或者通过 this 标识符来引用：
```
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});
```

许多 context 的访问器和方法为了便于访问和调用，简单的委托给他们的 ctx.request 和 ctx.response 所对应的等价方法， 比如说 ctx.type 和 ctx.length 代理了 response 对象中对应的方法，ctx.path 和 ctx.method 代理了 request 对象中对应的方法。

### ctx对象里面的API们

ctx.req

Node 的 request 对象。

ctx.res

Node 的 response 对象。

Koa 不支持 直接调用底层 res 进行响应处理。请避免使用以下 node 属性：

res.statusCode
res.writeHead()
res.write()
res.end()
ctx.request

Koa 的 Request 对象。

ctx.response

Koa 的 Response 对象。

ctx.app

应用实例引用。

ctx.cookies.get(name, [options])

获得 cookie 中名为 name 的值，options 为可选参数：

'signed': 如果为 true，表示请求时 cookie 需要进行签名。
注意：Koa 使用了 Express 的 cookies 模块，options 参数只是简单地直接进行传递。

ctx.cookies.set(name, value, [options])

设置 cookie 中名为 name 的值，options 为可选参数：

signed: 是否要做签名
expires: cookie 有效期时间
path: cookie 的路径，默认为 /'
domain: cookie 的域
secure: false 表示 cookie 通过 HTTP 协议发送，true 表示 cookie 通过 HTTPS 发送。
httpOnly: true 表示 cookie 只能通过 HTTP 协议发送
注意：Koa 使用了 Express 的 cookies 模块，options 参数只是简单地直接进行传递。

ctx.throw(msg, [status])

抛出包含 .status 属性的错误，默认为 500。该方法可以让 Koa 准确的响应处理状态。 Koa支持以下组合：

this.throw(403)
this.throw('name required', 400)
this.throw(400, 'name required')
this.throw('something exploded')
this.throw('name required', 400) 等价于：

var err = new Error('name required');
err.status = 400;
throw err;
注意：这些用户级错误被标记为 err.expose，其意味着这些消息被准确描述为对客户端的响应，而并非使用在您不想泄露失败细节的场景中。

ctx.respond

为了避免使用 Koa 的内置响应处理功能，您可以直接赋值 this.repond = false;。如果您不想让 Koa 来帮助您处理 reponse，而是直接操作原生 res 对象，那么请使用这种方法。

注意： 这种方式是不被 Koa 支持的。其可能会破坏 Koa 中间件和 Koa 本身的一些功能。其只作为一种 hack 的方式，并只对那些想要在 Koa 方法和中间件中使用传统 fn(req, res) 方法的人来说会带来便利。

Request aliases

以下访问器和别名与 Request 等价：
```
ctx.header
ctx.method
ctx.method=
ctx.url
ctx.url=
ctx.originalUrl
ctx.path
ctx.path=
ctx.query
ctx.query=
ctx.querystring
ctx.querystring=
ctx.host
ctx.hostname
ctx.fresh
ctx.stale
ctx.socket
ctx.protocol
ctx.secure
ctx.ip
ctx.ips
ctx.subdomains
ctx.is()
ctx.accepts()
ctx.acceptsEncodings()
ctx.acceptsCharsets()
ctx.acceptsLanguages()
ctx.get()
```

Response aliases

以下访问器和别名与 Response 等价：
```
ctx.body
ctx.body=
ctx.status
ctx.status=
ctx.length=
ctx.length
ctx.type=
ctx.type
ctx.headerSent
ctx.redirect()
ctx.attachment()
ctx.set()
ctx.remove()
ctx.lastModified=
ctx.etag=
```

## 相关学习资料

GitHub repository：https://github.com/koajs
Examples：https://github.com/koajs/examples
Wiki：https://github.com/koajs/koa/wiki
Guide：https://github.com/koajs/koa/blob/master/docs/guide.md
