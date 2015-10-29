# .jshintrc配置文件说明

## 严格选项

```JavaScript

// 禁用位运算符，位运算符在 JavaScript 中使用较少，经常是把 && 错输成 &
"bitwise": true

// 循环或者条件语句必须使用花括号包围
"curly": true

// 强制使用三等号
"eqeqeq": true

// 兼容低级浏览器 IE 6/7/8/9
"es3": true

// 禁止重写原生对象的原型，比如 Array ， Date
"freeze": true

// 代码缩进
"indent": false

// 禁止定义之前使用变量，忽略 function 函数声明
"latedef": "nofunc"

// 构造器函数首字母大写
"newcap": true

// 禁止使用 arguments.caller 和 arguments.callee
"noarg":true

// 值为 true 时，禁止单引号和双引号混用
"quotmark": false

// 变量未定义
"undef": true

// 变量未使用
"unused": true

// 严格模式
"strict":true

// 最多参数个数
"maxparams": 4

// 最大嵌套深度
"maxdepth": 4

// 复杂度检测
"maxcomplexity":true

// 最大行数
"maxlen": 600

```

## 宽松选项

```javascript

// 控制“缺少分号”的警告
"asi": true
"boss": true

// 忽略 debugger
"debug": true

// 控制 eval 使用警告
"evil": true

// 检查一行代码最后声明后面的分号是否遗漏
"lastsemic": true

// 检查不安全的折行，忽略逗号在最前面的编程风格
"laxcomma": true

// 检查循环内嵌套 function
"loopfunc": true

// 检查多行字符串
"multistr": true

// 检查无效的 typeof 操作符值
"notypeof": true

// person['name'] vs. person.name
"sub": true

// new function () { ... } 和 new Object ;
"supernew": true

// 在非构造器函数中使用 this
"validthis": true
```
## 环境

```javascript
// 预定义全局变量 document ， navigator ， FileReader 等
"browser": true

// 定义用于调试的全局变量： console ， alert
"devel": true

// 定义全局变量
"jquery": true,
"node": true

```
