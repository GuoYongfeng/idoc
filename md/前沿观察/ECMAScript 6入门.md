# ECMAScript 6入门

> 注：文章内容来自阮一峰的《ECMAScript 6入门》

ECMAScript 6（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

标准的制定者有计划，以后每年发布一次标准，使用年份作为标准的版本。因为当前版本的ES6是在2015年发布的，所以又称ECMAScript 2015。也就是说，ES6就是ES2015，下一年应该会发布小幅修订的ES2016。

## 安装与解析

各大浏览器的最新版本，对ES6的支持可以查看kangax.github.io/es5-compat-table/es6/。随着时间的推移，支持度已经越来越高了，ES6的大部分特性都实现了。

### node环境

Node.js是JavaScript语言的服务器运行环境，对ES6的支持度比浏览器更高。通过Node，可以体验更多ES6的特性。

使用下面的命令，可以查看Node所有已经实现的ES6特性。
```
$ node --v8-options | grep harmony
```
### 浏览器环境

使用的ES6转码器，可以将ES6代码转为ES5代码，从而在浏览器或其他环境执行。这意味着，你可以用ES6的方式编写程序，又不用担心现有环境是否支持。

你可以使用以下工具将es6代码解析后在浏览器中加载
- Babel转码器
- Traceur转码器

或者在你的页面中嵌入解析文件运行es6的代码，比如使用babel的时候
```
<script src="node_modules/babel-core/browser.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```
## let和const命令

### let命令
### 块级作用域
### const命令
### 跨模块常量
### 全局对象的属性

## 对象的扩展

## 二进制数组

## Set和Map数据结构

## Generator函数

## Promise对象

## 异步操作和Async函数

## Class

## Decorator

## Module

### 严格模式
### export命令
### import命令
### 模块的整体加载
### module命令
### export default命令
### 模块的继承
### ES6模块的转码
