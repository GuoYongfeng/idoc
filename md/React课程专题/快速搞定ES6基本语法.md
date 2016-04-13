
<h1 style="font-size: 40px;text-align:center;color: blue;">快速搞定ES6基本语法</h1>

在项目中80%的时间用到的ES6语法只占其20%，所以我们暂时先集中精力把这20%学好，那就差不多够用了，剩下的可以看书或是查文档，现学现用。

**重要提示：**教程的示例代码请前往[es6-demo](https://github.com/GuoYongfeng/es6-demo)，下载后可以结合这个讲义进行学习操作。


## 1.let && const
let和const的出现让js有了块级作用域，这个块级作用域是个神器，由于之前没有块级作用域的存在，以及var关键字的变量提升，导致我们调试的时候会出现一些莫名其妙的问题，同时也是很长一段时间面试的常问问题之一。

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

## 2.destructuring

destructuring是解构的意思，ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。来两个例子看看大家就明白了。

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

## 3.template string

```
let name = 'guoyongfeng';
let age = 18;

console.log(`${name} want to drink ${age}`)
```

## 4.arrow function

arrow function译名即为箭头函数，这早已不是一个新名词，下面简单看一下它的写法
```
class Animal {
    constructor(name){
        this.name = name;
    }
    // type = 'water'这里的意思是当type未传值的时候默认是'water'
    drink(type = 'water'){
	    // 使用了箭头函数
        setInterval( () => {
	        // 模板字符串
            console.log(`${this.name} want to drink ${type}`)
        }, 1000)
    }
}

let pig = new Animal('pig');

console.log(pig.drink('milk'));

export default Animal;

```

## 5.rest

我们知道JS函数内部有个arguments对象，可以拿到全部实参。现在ES6给我们带来了一个新的对象，可以拿到除开始参数外的参数，即剩余参数，听起来好屌的样子，我们来段代码。

```
// rest
function restFunc(a, ...rest) {
  console.log(a)
  console.log(rest)
}
restFunc(1);
restFunc(1, 2, 3, 4);
```
## 6.spread

spread为扩展操作符，来一个有点无聊的例子。
```
var args = ["a", "b", "c"];
console.log(...args); // "a" "b" "c"
```
就是这样无聊的例子，如果没有spread语法的支持，我们还得来个遍历，ES6委员会真是程序员的小棉袄。


## 7.class, extends, super

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

而现在我们可以使用class来定义一个类
代码清单：`class.js`
```
'use strcit';

export default class Point {
  // constructor方法是类的默认方法
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  // 这里定义的都是类的公共方法
  // 等同于Point.prototype.toString
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var point = new Point(2, 3);

console.log(point.toString());
console.log(point.hasOwnProperty('x'))
console.log(point.hasOwnProperty('y'))
console.log(point.hasOwnProperty('toString'))
console.log(point.__proto__.hasOwnProperty('toString'))
```

在moduleB中来import这个文件进行运行。
代码清单：`moduleB.js`
```
'use strict';

// import moduleA from './moduleA';
import Point from './class.js';
```

## 8.Object.assign
Object.assign用于对象的合并，ES6对object做了很多扩展，assign是最值得点评的。想必你很熟悉jquery提供的extend接口，那么ES6的Object.assign就从语法层面做了这件事情，是不是很nice。

```
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
console.log(target); // {a:1, b:2, c:3}
```

## 9.Decorator
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


## 10.promise
Promise是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了Promise对象。

下面摘抄一个《ECMASCRIPT 6入门》书上的例子，使用promise模拟一个ajax方法的demo用于大家理解
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

## 11.export, import

以前我们学习AMD和CMD规范，讨论用什么样的模块化规范比较合适，更有甚者，不得不使用UMD规范来做全兼容，现在好了，ES6在语言层面推出了模块化的写法。

新建两个模块
```
$ touch moduleA.js moduleB.js
```

代码清单：`moduleA.js`
```
'use strict';

export default function foo(x, y) {
  return x * y;
}

// 另外几种导出的方式
// function foo() {
//   console.log('foo');
// }
// export { foo as default };
// export var a = 1;
// export const PI = 3.14;
```

代码清单：`moduleB.js`
```
'use strict';

import moduleA from './moduleA';

console.log( moduleA(2, 3) );
```

代码清单：`index.js`
```
'use strict';

import moduleB from './util/moduleB ';
```

最终运行打印出结果：6；



## 结语

多的不在赘述，以上的一些知识可以当成学习ES6的一个快餐，不成体系，但是快速掌握常用的一些语法，有时间有精力建议最好买本书通读一遍。
