# 轻型MVVM框架Vue.js

## 介绍

Vue.js 是一个用于创建 web 交互界面的库。

从技术角度讲，Vue.js 专注于 MVVM 模型的 ViewModel 层。它通过双向数据绑定把 View 层和 Model 层连接了起来。实际的 DOM 封装和输出格式都被抽象为了 Directives 和 Filters。

从哲学角度讲，Vue 希望通过一个尽量简单的 API 来提供响应式的数据绑定和可组合、复用的视图组件。它不是一个大而全的框架——它只是一个简单灵活的视图层。您可以独立使用它快速开发原型、也可以混合别的库做更多的事情。它同时和诸如 Firebase 这一类的 BaaS 服务有着天然的契合度。

Vue.js 的 API 设计深受 AngularJS、KnockoutJS、Ractive.js 和 Rivets.js 的影响。尽管有不少相似之处，但我们相信 Vue.js 能够在简约和功能之间的微妙平衡中体现出其独有的价值。

即便您已经熟悉了一些这类的库或框架，我们还是推荐您继续阅读接下来的概览，因为您对它们的认识也许和它们在 Vue.js 语境下的定义不尽相同。

## 安装
- CDN
```
<script src="https://raw.githubusercontent.com/yyx990803/vue/0.12.16/dist/vue.min.js"><script>
```
- npm
- bower

## 概念综述

### ViewModel

一个同步 Model 和 View 的对象。在 Vue.js 中，每个 Vue 实例都是一个 ViewModel。它们是通过构造函数 Vue 或其子类被创建出来的。
```
var vm = new Vue({ /* options */ })
```
这是您作为一个开发者在使用 Vue.js 时主要打交道的对象。更多的细节请移步至 Vue 构造函数。

### 视图 View

被 Vue 实例管理的 DOM 节点。
```
vm.$el // The View
```
Vue.js 使用基于 DOM 的模板。每个 Vue 实例都关联着一个相应的 DOM 元素。当一个 Vue 实例被创建时，它会递归遍历根元素的所有子结点，同时完成必要的数据绑定。当这个视图被编译之后，它就会自动响应数据的变化。

在使用 Vue.js 时，除了自定义指令 (稍后会有解释)，您几乎不必直接接触 DOM。当数据发生变化时，视图将会自动触发更新。这些更新的粒度精确到一个文字节点。同时为了更好的性能，这些更新是批量异步执行的。

### 模型 Model

一个轻微改动过的原生 JavaScript 对象。
```
vm.$data // The Model
```

Vue.js 中的模型就是普通的 JavaScript 对象 —— 也可以称为数据对象。一旦某对象被作为 Vue 实例中的数据，它就成为一个 “响应式” 的对象了。你可以操作它们的属性，同时正在观察它的 Vue 实例也会收到提示。Vue.js 把数据对象的属性都转换成了 ES5 中的 getter/setters，以此达到无缝的数据观察效果：无需脏值检查，也不需要刻意给 Vue 任何更新视图的信号。每当数据变化时，视图都会在下一帧自动更新。

Vue 实例代理了它们观察到的数据对象的所有属性。所以一旦一个对象 { a: 1 } 被观察，那么 vm.$data.a 和 vm.a 将会返回相同的值，而设置 vm.a = 2 则也会修改 vm.$data。

数据对象是被就地转化的，所以根据引用修改数据和修改 vm.$data 具有相同的效果。这也意味着多个 Vue 实例可以观察同一份数据。在较大型的应用程序中，我们也推荐将 Vue 实例作为纯粹的视图看待，同时把数据处理逻辑放在更独立的外部数据层。

值得提醒的是，一旦数据被观察，Vue.js 就不会再侦测到新加入或删除的属性了。作为弥补，我们会为被观察的对象增加 $add , $set和 $delete 方法。

下面是对 Vue.js 数据观测机制实现的高层概览：

数据观察

### 指令 Directives

带特殊前缀的 HTML 特性，可以让 Vue.js 对一个 DOM 元素做各种处理。
```
<div v-text="message"></div>
```

这里的 div 元素有一个 v-text 指令，其值为 message。Vue.js 会让该 div 的文本内容与 Vue 实例中的 message 属性值保持一致。

Directives 可以封装任何 DOM 操作。比如 v-attr 会操作一个元素的特性；v-repeat 会基于数组来复制一个元素；v-on 会绑定事件等。稍后会有更多的介绍。

Mustache 风格绑定

你也可以使用 mustache 风格的绑定，不管在文本中还是在属性中。它们在底层会被转换成 v-text 和 v-attr 的指令。比如：
```
<div id="person-{{id}}">Hello {{name}}!</div>
```

这很方便，不过有一些注意事项：

一个 <image> 的 src 属性在赋值时会产生一个 HTTP 请求，所以当模板在第一次被解析时会产生一个 404 请求。这种情况下更适合用 v-attr。

Internet Explorer 在解析 HTML 时会移除非法的内联 style 属性，所以如果你想支持 IE，请在绑定内联 CSS 的时候始终使用 v-style。

你可以使用三对花括号来回避 HTML 代码，而这种写法会在底层转换为 v-html：

```
{{{ safeHTMLString }}}
```

不过这种用法会留下 XSS 攻击的隐患，所以建议只对绝对信任的数据来源使用三对花括号的写法，或者先通过自定义的过滤器 (filter) 对不可信任的 HTML 进行过滤。

最后，你可以在你的 mustache 绑定里加入 * 来注明这是一个一次性撰写的数据，这样的话它就不会响应后续的数据变化：
```
{{* onlyOnce }}
```

### 过滤器 Filters

过滤器是用于在更新视图之前处理原始值的函数。它们通过一个“管道”在指令或绑定中进行处理：
```
<div>{{message | capitalize}}</div>
```

这样在 div 的文本内容被更新之前，message 的值会先传给 capitalizie 函数处理。更多内容可移步至 深入了解过滤器 (Filters)。

### 组件 Components

Component Tree

在 Vue.js，每个组件都是一个简单的 Vue 实例。一个树形嵌套的各种组件就代表了你的应用程序的各种接口。通过 Vue.extend 返回的自定义构造函数可以把这些组件实例化，不过更推荐的声明式的用法是通过 Vue.component(id, constructor) 注册这些组件。一旦组件被注册，它们就可以在 Vue 实例的模板中以自定义元素形式使用了。

```
<my-component>
  <!-- internals handled by my-component -->
</my-component>
```

这个简单的机制使得我们可以以类似 Web Components 的声明形式对 Vue 组件进行复用和组合，同时无需最新版的浏览器或笨重的 polyfills。通过将一个应用程序拆分成简单的组件，代码库可以被尽可能的解耦，同时更易于维护。更多关于组件的内容，请移步至 组件系统。

### 简单示例

```
<div id="demo">
  <h1>{{title | uppercase}}</h1>
  <ul>
    <li
      v-repeat="todos"
      v-on="click: done = !done"
      class="{{done ? 'done' : ''}}">
      {{content}}
    </li>
  </ul>
</div>

var demo = new Vue({
  el: '#demo',
  data: {
    title: 'todos',
    todos: [
      {
        done: true,
        content: 'Learn JavaScript'
      },
      {
        done: false,
        content: 'Learn Vue.js'
      }
    ]
  }
})
```

## 指令

## 过滤器

## 列表渲染

## 事件监听

## 处理表单

## 计算属性

## 自定义指令

## 自定义过滤器

## 组件系统

## 过渡效果

## 构建应用
