<h1 style="font-size: 40px;text-align:center;color: #007cdc;">快速搞定常用的ES6新特性</h1>

在项目中80%的时间用到的ES6语法只占其20%，所以我们暂时先集中精力把这20%学好，那就差不多够用了，剩下的可以看书或是查文档，现学现用。

**重要提示：**教程的示例代码请前往[es6-demo](https://github.com/GuoYongfeng/es6-demo)，下载后可以结合这个讲义进行学习操作。

## 1. Let + Const 块级作用域和常量

let和const的出现让 JS 有了块级作用域，还可以像强类型语言一样定义常量。由于之前没有块级作用域以及 var 关键字所带来的变量提升，经常给我们的开发带来一些莫名其妙的问题。

下面看两个简单的demo理解。
```
// demo 1
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}

// demo 2
const PI = 3.1415;
console.log(PI); // 3.1415

PI = 3;
console.log(PI); // TypeError: "PI" is read-only
```


## 2.Arrows 箭头函数

- 箭头函数简化了函数的的定义方式，一般以 "=>" 操作符左边为输入的参数，而右边则是进行的操作以及返回的值Inputs=>outputs。
- 箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。正是因为它没有this，所以也就不能用作构造函数，从而避免了this指向的问题。

请看下面的例子。

```
var array = [1, 2, 3];
//传统写法
array.forEach(function(v, i, a) {
    console.log(v);
});
//ES6
array.forEach(v = > console.log(v));
```

更多示例：

```
const Template = {
    test: function(){
        console.log(this);
        $('#event').on('click',()=>{
            // 大家觉得这个 this 是什么
            console.log(this);
        });
    }
};
Template.test();

```

## 3.Class, extends, super 类的支持

回想之前，如果我们需要模拟一个js的类，一般会采用构造函数加原型的方式。
```
function Point(x,y){
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
}
```

ES6中添加了对类的支持，引入了class关键字（其实class在JavaScript中一直是保留字，目的就是考虑到可能在以后的新版本中会用到，现在终于派上用场了）。

JS本身就是面向对象的，ES6中提供的类实际上只是JS原型模式的包装。现在提供原生的class支持后，对象的创建，继承更加直观了，并且父类方法的调用，实例化，静态方法和构造函数等概念都更加形象化。

下面代码展示了类在ES6中的使用。

```
//类的定义
class Animal {
	//ES6中新型构造器
    constructor(name) {
        this.name = name;
    }
    //实例方法
    sayName() {
        console.log('My name is '+this.name);
    }
}
//类的继承
class Programmer extends Animal {
    constructor(name) {
    	//直接调用父类构造器进行初始化
        super(name);
    }
    program() {
        console.log("I'm coding...");
    }
}

//测试我们的类
var animal=new Animal('dummy'),
zf=new Programmer('zf');

animal.sayName();//输出 ‘My name is dummy’
zf.sayName();//输出 ‘My name is zf’
zf.program();//输出 ‘I'm coding...’
```

## 4.Enhanced Object Literals 增强的对象字面量

对象字面量被增强了，写法更加简洁与灵活，同时在定义对象的时候能够做的事情更多了。具体表现在：

- 可以在对象字面量里面定义原型
- 定义方法可以不用function关键字
- 直接调用父类方法

这样一来，对象字面量与前面提到的类概念更加吻合，在编写面向对象的JavaScript时更加轻松方便了。

```
//通过对象字面量创建对象
var human = {
    breathe() {
        console.log('breathing...');
    }
};
var worker = {
    __proto__: human, //设置此对象的原型为human,相当于继承human
    company: 'freelancer',
    work() {
        console.log('working...');
    }
};
human.breathe();//输出 ‘breathing...’
//调用继承来的breathe方法
worker.breathe();//输出 ‘breathing...’
```

## 5.Template Strings 字符串模板

字符串模板相对简单易懂些。ES6中允许使用反引号 ` 来创建字符串，此种方法创建的字符串里面可以包含由美元符号加花括号包裹的变量${vraible}。如果你使用过像C#等后端强类型语言的话，对此功能应该不会陌生。

```
//产生一个随机数
var num = Math.random();
//将这个数字输出到console
console.log(`your num is ${num}`);

let name = 'guoyongfeng';
let age = 18;

console.log(`${name} was ${age}`)

```
## 6.Destructuring 解构

Destructuring是解构的意思，ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

比如若一个函数要返回多个值，常规的做法是返回一个对象，将每个值做为这个对象的属性返回。但在ES6中，利用解构这一特性，可以直接返回一个数组，然后数组中的值会自动被解析到对应接收该值的变量中。

```
var [x,y]=getVal(),//函数返回值的解构
    [name,,age]=['zf','male','secrect'];//数组解构

function getVal() {
    return [ 1, 2 ];
}

console.log('x:'+x+', y:'+y);//输出：x:1, y:2
console.log('name:'+name+', age:'+age);//输出： name:zf, age:secrect
```

数组、对象和字符串的解构赋值示例：

```
'use strict';

// 数组的解构赋值
let [foo, [[bar], baz]] = [1, [[2], 3]];
console.log(foo); // 1
console.log(bar); // 2
console.log(baz); // 3

// 对象的解构赋值
var { foo, bar } = { foo: "aaa", bar: "bbb" };
console.log(foo);   // "aaa"
console.log(bar );  // "bbb"

// 字符串的解构赋值
const [a, b, c, d, e] = 'hello';
console.log(a + b + c + e); // 'hello'

```

## 7.Default + Rest + Spread

### Default 默认参数值

现在可以在定义函数的时候指定参数的默认值了，而不用像以前那样通过逻辑或操作符来达到目的了。

```
function sayHello(name){
	//传统的指定默认参数的方式
	var name=name||'dude';
	console.log('Hello '+name);
}
//运用ES6的默认参数
function sayHello2(name='dude'){
	console.log(`Hello ${name}`);
}
sayHello();//输出：Hello dude
sayHello('zf');//输出：Hello zf
sayHello2();//输出：Hello dude
sayHello2('zf');//输出：Hello zf
```

### Rest 剩余参数

不定参数是在函数中使用命名参数同时接收不定数量的未命名参数。这只是一种语法糖，在以前的JavaScript代码中我们可以通过 `arguments` 变量来达到这一目的。

不定参数的格式是三个句点后跟代表所有不定参数的变量名。比如下面这个例子中，…x代表了所有传入add函数的参数。

一个简单示例：
```
// rest
function restFunc(a, ...rest) {
  console.log(a)
  console.log(rest)
}
restFunc(1);
restFunc(1, 2, 3, 4);
```

再看一个：
```
//将所有参数相加的函数
function add(...x){
	return x.reduce((m,n)=>m+n);
}
//传递任意个数的参数
console.log(add(1,2,3));//输出：6
console.log(add(1,2,3,4,5));//输出：15
```

### Spread  扩展操作符

扩展操作符则是另一种形式的语法糖，它允许传递数组或者类数组直接做为函数的参数而不用通过apply。

```
var people=['zf','John','Sherlock'];

function sayHello(people1,people2,people3){
	console.log(`Hello ${people1},${people2},${people3}`);
}
//但是我们将一个数组以拓展参数的形式传递，它能很好地映射到每个单独的参数
sayHello(...people);//输出：Hello zf,John,Sherlock

//而在以前，如果需要传递数组当参数，我们需要使用函数的apply方法
sayHello.apply(null,people);//输出：Hello zf,John,Sherlock
```

## 8.Map，Set 和 WeakMap，WeakSet

这些是新加的集合类型，提供了更加方便的获取属性值的方法，不用像以前一样用`hasOwnProperty`来检查某个属性是属于原型链上的呢还是当前对象的。同时，在进行属性值添加与获取时有专门的`get，set `方法。

```
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;
```

有时候我们会把对象作为一个对象的键用来存放属性值，普通集合类型比如简单对象会阻止垃圾回收器对这些作为属性键存在的对象的回收，有造成内存泄漏的危险。而`WeakMap,WeakSet`则更加安全些，这些作为属性键的对象如果没有别的变量在引用它们，则会被回收释放掉，具体还看下面的例子。

```
// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });//因为添加到ws的这个临时对象没有其他变量引用它，所以ws不会保存它的值，也就是说这次添加其实没有意思
```

## 9.Proxies

Proxy可以监听对象身上发生了什么事情，并在这些事情发生后执行一些相应的操作。一下子让我们对一个对象有了很强的追踪能力，同时在数据绑定方面也很有用处。

```
//定义被侦听的目标对象
var engineer = { name: 'Joe Sixpack', salary: 50 };

//定义处理程序
var interceptor = {
  set: function (receiver, property, value) {
    console.log(property, 'is changed to', value);
    receiver[property] = value;
  }
};

//创建代理以进行侦听
engineer = Proxy(engineer, interceptor);
//做一些改动来触发代理
engineer.salary = 60;//控制台输出：salary is changed to 60
```

上面代码我已加了注释，这里进一步解释。对于处理程序，是在被侦听的对象身上发生了相应事件之后，处理程序里面的方法就会被调用，上面例子中我们设置了`set`的处理函数，表明，如果我们侦听的对象的属性被更改，也就是被`set`了，那这个处理程序就会被调用，同时通过参数能够得知是哪个属性被更改，更改为了什么值。

## 10.Object assign

Object.assign 用于对象的合并，ES6对object做了很多扩展，assign是最值得点评的。想必你很熟悉jquery提供的extend接口，那么ES6的Object.assign就是从语法层面做了这件事情，是不是很nice。

```
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
console.log(target); // {a:1, b:2, c:3}
```


## 11.Promises

ES6的Promise是一个非常重要的特性，它是处理异步操作的一种模式。有了它，JavaScript异步嵌套的问题算是得到了比较好的解决。同时，Promise也是ES7中async/await的基础。

我们来看ES6中的Promise对象的使用：

```
//创建promise
var promise = new Promise(function(resolve, reject) {
    // 进行一些异步或耗时操作
    if ( /*如果成功 */ ) {
        resolve("Stuff worked!");
    } else {
        reject(Error("It broke"));
    }
});
//绑定处理程序
promise.then(function(result) {
	//promise成功的话会执行这里
    console.log(result); // "Stuff worked!"
}, function(err) {
	//promise失败会执行这里
    console.log(err); // Error: "It broke"
});
```

使用promise模拟一个ajax方法的demo用于大家理解
```
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if ( this.readyState !== 4 ) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

## 12.Decorator
修饰器（Decorator）是一个表达式，用来修改类的行为。这是ES7的一个提案，目前Babel（babel-plugin-transform-decorators-legacy）转码器已经支持。

不知道大家有没有使用过java的spring mvc，其中的注解就跟这个比较相似，学习React的话可以重点关注下这个语法，因为后面使用redux类库的时候会频繁的用到decorator。

首先说下如何配置babel的插件来进行decorator的解析。
```
// 官网提供了babel-plugin-transform-decorators这个插件来解析，但是我发现不work，就找了下面这个
$ npm install babel-plugin-transform-decorators-legacy --save-dev
```
配置`.babelrc`的plugins字段。
```
{
  "presets": ["es2015", "react", "stage-0"],
  "plugins": ["transform-decorators-legacy"]
}

```

ok，接下来来段使用decorator的示例代码
```
function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass {}

console.log(MyTestableClass.isTestable) // true
```


## 13.Modules 模块

在ES6标准中，JavaScript原生支持module了。这种将JS代码分割成不同功能的小块进行模块化的概念是在一些三方规范中流行起来的，比如CommonJS和AMD模式。

将不同功能的代码分别写在不同文件中，各模块只需导出公共接口部分，然后通过模块的导入的方式可以在其他地方使用。

不过，还是有很多细节的地方需要注意，我们看例子：

简单使用方式：
```
// point.js
export class Point {
     constructor (x, y) {
         public x = x;
         public y = y;
     }
 }


// myapp.js
//这里可以看出，尽管声明了引用的模块，还是可以通过指定需要的部分进行导入
import Point from "point";

var origin = new Point(0, 0);
console.log(origin);
```

### export

```
// demo1：简单使用
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

// 等价于
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

```
// demo2：还可以这样
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

```
// demo3：需要注意的是

// 报错
function f() {}
export f;

// 正确
export function f() {};

```

我们再来看一下export的默认输出：

```
export default function () {
  console.log('foo');
}
```

为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到export default命令，为模块指定默认输出。这样其他模块加载该模块时，import命令可以为该匿名函数指定任意名字。

需要注意的是，这时import命令后面，不使用大括号。


最后需要强调的是：ES6模块加载的机制，与CommonJS模块完全不同。CommonJS模块输出的是一个值的拷贝，而ES6模块输出的是值的引用。

- CommonJS模块输出的是被输出值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。

- ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令import时，不会去执行模块，而是只生成一个动态的只读引用。等到真的需要用到时，再到模块里面去取值，换句话说，ES6的输入有点像Unix系统的”符号连接“，原始值变了，import输入的值也会跟着变。因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。


### import

```
// 1
import $ from 'jquery';

// 2
import {firstName, lastName, year} from './profile';

// 3
import React, { Component, PropTypes } from 'react';

// 4
import * as React from 'react';

```

## 结语

多的不在赘述，以上的一些知识可以当成学习ES6的一个快餐，不成体系，希望能够帮助你快速的掌握一些ES6常用的语法，有时间有精力建议买本阮一峰老师的《ES6入门》通读一遍。
