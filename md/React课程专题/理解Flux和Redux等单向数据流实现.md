# 理解Redux单向数据流实现

为什么要有这个教程呢？

在我尝试学习 Redux 的时候，我发现之前的阅读过的一些文章加上个人的经验，
让我对 flux 产生了一些错误的认知。
当然，我不是说那些关于 flux 的文章写得不好，只是我没能正确地领会其中的概念。
到头来，我只是对着各种 flux 框架（Reflux、Flummox、FB Flux）的文档照猫画虎，
并试着把它们和之前了解到的理论概念联系起来。
等我用了 Redux 之后，我才发现原来flux比我想象的要简单很多。
这些都归功于 Redux 通过优良的设计来减少样板文件，而其它框架则是为了减少样板文件却又引入了很多新的代码。
我现在觉得用通过 Redux 来学习 flux 比通过其他框架高效得多。
这就是为什么我想分享给大家，用我自己的话来说，
通过关注 Redux 的用法来理解 flux 的概念。

你一定已经看过这张著名的 flux 的单向数据流图了。

<img src="/img/redux/flux.png" />

在这个教程里，我们会一步步地向你介绍上图里的各个概念。
我们会把这些概念分成单独的章节来介绍它们存在的意义和作用，
而不会笼统地介绍整张数据流图。
在最后，当我们理解了每一个概念后，我们会发现这张图真是意义深远啊！

在我们开始之前，我们先聊下一flux存在的意义以及我们为什么需要它。
假设我们正在构建一个网站应用，那么这个网站应用会由什么组成呢？
1) 模板/HTML = View
2) 填充视图的数据 = Model
3) 获取数据，将所有视图组装在一起，
  用户事件数据变化时的响应，等等的逻辑 = Controller

这就是个非常典型的 MVC，而且它和flux的概念看起来很像的，
只是在某些表述上有些小小的不同：
- Model 看起来像 Store
- 用户事件、数据变化以及它们的处理程序看起来像
 "action creators" -> action -> dispatcher -> callback
- View 看起来像 React views (或者其他什么的)

所以，flux 就只是一个新名词么？不全是，但是新名词是很重要的，
因为通过引入这些新术语我们可以更准确地表述各种专业术语。
举一个例子，获取数据是一个 action，一个点击是一个 action，
一个input变化也是一个action等等。我们都已经习惯了从我们的应用里分发 action，
只是以不同的方式称呼它们。 不同于直接修改 Model 和 View，
Flux 确保所有 action 首先通过一个dispatcher，
然后再是store，最后通知所有的store观测者。

为了弄清楚MVC和flux的不同，
我们举一个典型的MVC应用的用例：
一个典型的MVC应用的流程大致上是这样的：
1) 用户点击按钮“A”
2) 点击按钮“A”的处理程序触发 Model “A”的改变
3) Model “A”的改变处理程序触发 Model “B”的改变
4) Model “B”的改变处理程序触发 View “B”的改变并重新渲染自身

在这样的一个环境里，当应用出错的时候快速地定位 bug 来源是一件非常困难的事情。
这是因为每个 View 可以监视任何的 Model，
并且每个 Model 可以监视其它所有 Model，所以数据会从四面八方涌来，并且被许多源（view 或者 model）改变。

当我们用 flux 以及它的单向数据流的时候，上面的例子就会变成这样子：
1) 用户点击按钮“A”
2) 点击按钮“A”的处理程序会触发一个被分发的 action，并改变 Store “A”
3) 因为其它的 Store 也被这个 action 通知了，所以 Store “B”也会对相同的 action 做出反应
4) View “B”因为 Store  A 和 Store B的改变而收到通知，并重新渲染

来看一下我们是如何避免 Store A 和 Store B 直接相关联的。
每一个 Store 只能被一个 action 修改，别无他选。
并且当所有 Store 回应了 action 后，最终所有 View 都会更新。由此可见，数据总是沿一个方向进行流动：
```
   action -> store -> view -> action -> store -> view -> action -> ...
```
就像我们从一个 action 开始我们的用例，
让我们以 action 和 action creator 来开始我们的教程。

## 1.simple-action-creator

我们在前言中已经简单提到过 action，但具体什么是 "action creator"，它们又是如何关联到 "action" 的呢？

其实，通过几行简单的代码就可以解释清楚了！

action creator 就是函数而已...
```
var actionCreator = function() {
   ...负责构建一个 action （是的，action creator 这个名字已经很明显了）并返回它
  return {
      type: 'AN_ACTION'
  }
}
```

这就完了？是的，就是这样。

然而，有一件事情需要注意，那就是 action 的格式。在 flux 中，一般约定 action 是一个拥有 “type” 属性的对象。
这个 “type” 在以后处理 action 时会用到。当然，action 依旧可以拥有其他属性，你可以任意存放想要的数据。

在后面的章节中，我们会发现 action creator 实际上可以返回 action 以外的其他东西，比如一个函数。
这在异步 action 处理中极为有用（更多的内容可以查阅 dispatch-async-action.js）。

我们可以直接调用 action creator，如同预期的一样，我们会得到一个 action：
```
console.log(actionCreator())
```
输出： { type: 'AN_ACTION' }

好了，以上代码没有任何问题，但是，啥用也没有...
在实际的场景中，我们需要的是将 action 发送到某个地方，让任何关心它的人知道发生了什么，并且做出相应的处理。
我们将这个过程称之为“分发 action（Dispatching an action）”。

为了分发 action，我们需要...一个分发函数（=￣ω￣=）。
并且，为了让任何对它感兴趣的人可以感知到 action 被发起，我们还需要一个注册“处理器（handlers）”的机制。
这些 action 的“处理器”在传统的 flux 应用中被称为 store，在下个章节中，我们会了解到它们在 redux 中被称为什么。

到目前为止，我们的应用中包含了以下流程：
```
ActionCreator -> Action
```
可以在以下链接中了解更多关于 action 和 action creator 的内容：

http:rackt.org/redux/docs/recipes/ReducingBoilerplate.html

## 2.about-state-and-meet-redux

有时我们在应用程序中所处理的动作不仅告知我们发生了什么事，而且告诉我们需要随之更新数据。

事实上对于任何应用而言处理这些动作都显得非常棘手。
如何在应用程序的整个生命周期内维持所有数据？
如何修改这些数据？
如何把数据变更传播到整个应用程序？

于是 Redux 登场。

Redux (https:github.com/rackt/redux) 是一个『可预测化状态的 JavaScript 容器』。

我们先回顾上述提出的问题并用 Redux 的词汇表给出以下解答（部分词汇也来源于 Flux）：

如何在应用程序的整个生命周期内维持所有数据？
    以你想要的方式维持这些数据，例如 JavaScript 对象、数组、不可变数据，等等。
    我们把应用程序的数据称为状态。这是有道理的，因为我们所说的数据会随着时间的推移发生变化，这其实就是应用的状态。
    但是我们把这些状态信息转交给了 Redux（还记得么？Redux 就是一个『容纳状态的容器』）。
如何修改这些数据？
    我们使用 reducer 函数修改数据（在传统的 Flux 中我们称之为 store）。
    Reducer 函数是 actions 的订阅者。
    Reducer 函数只是一个纯函数，它接收应用程序的当前状态以及发生的 action，然后返回修改后的新状态（或者有人称之为归并后的状态）。
如何把数据变更传播到整个应用程序？
    使用订阅者来监听状态的变更情况。

Redux 帮你把这些连接起来。
总之 Redux 提供了：
   1）存放应用程序状态的容器
   2）一种把 action 分发到状态修改器的机制，也就是 reducer 函数
   3）监听状态变化的机制

我们把 Redux 实例称为 store 并用以下方式创建：
```
  import { createStore } from 'redux'
  var store = createStore()
```
但是当你运行上述代码，你会发现以下异常消息：
```
   Error: Invariant Violation: Expected the reducer to be a function.
```
这是因为 createStore 函数必须接收一个能够修改应用状态的函数。

我们再试一下
```
import { createStore } from 'redux'

var store = createStore(() => {})
```
看上去没有问题了...

## 3.simple-reducer

现在，我们知道如何去创建一个 Redux 实例，并让它管理应用中的 state
下面讲一下这些 reducer 函数是如何转换 state 的。

Reducer 与 Store 区别：
你可能已经注意到，在简介章节中的 Flux 图表中，有 "Store"，但没有
Redux 中所拥有的 "Reducer"。那么，Store 与 Reducer 到底有哪些区别呢？
实际的区别比你想象的简单：Store 可以保存你的 data，而 Reducer 不能。
因此在传统的 Flux 中，Store 本身可以保存 state，但在 Redux 中，每次调用 reducer
时，都会传入待更新的 state。这样的话，Redux 的 store 就变成了
“无状态的 store” 并且改了个名字叫 Reducer。

如上所述，在创建一个 Redux 实例前，需要给它一个 reducer 函数...
```
import { createStore } from 'redux'

var store_0 = createStore(() => {})
```
... 因此每当一个 action 发生时，Redux 都会调用这个函数。
往 createStore 传 Reducer 的过程就是给 Redux 绑定 action 处理函数（也就是 Reducer）的过程。
action 处理函数在 01_simple-action-creator.js 章节中有讨论过。

在 Reducer 中打印一些 log
```
var reducer = function (...args) {
  console.log('Reducer was called with args', args)
}

var store_1 = createStore(reducer)
```
输出：Reducer was called with args [ undefined, { type: '@@redux/INIT' } ]

看出来了吗？实际上，我们的 reducer 被调用了，虽然我们没有 dispatch 任何 action...
这是因为在初始化应用 state 的时候，
Redux dispatch 了一个初始化的 action ({ type: '@@redux/INIT' })

在被调用时，一个 reducer 会得到这些参数：(state, action)
在应用初始化时，state 还没被初始化，因此它的值是 "undefined"，
这是非常符合逻辑的

在 Redux 发送 "init" action 之后，我们应用中的 state 又会是怎么样的呢？

## 4.get-state

How do we retrieve the state from our Redux instance?
```
import { createStore } from 'redux'

var reducer_0 = function (state, action) {
  console.log('reducer_0 was called with state', state, 'and action', action)
}

var store_0 = createStore(reducer_0)
Output: reducer_0 was called with state undefined and action { type: '@@redux/INIT' }
```
To get the state that Redux is holding for us, you call getState

console.log('store_0 state after initialization:', store_0.getState())
Output: store_0 state after initialization: undefined

So the state of our application is still undefined after the initialization? Well of course it is,
our reducer is not doing anything... Remember how we described the expected behavior of a reducer in
"about-state-and-meet-redux"?
   "A reducer is just a function that receives the current state of your application, the action,
   and returns a new state modified (or reduced as they call it)"
Our reducer is not returning anything right now so the state of our application is what
reducer() returns, hence "undefined".

Let's try to send an initial state of our application if the state given to reducer is undefined:
```
var reducer_1 = function (state, action) {
  console.log('reducer_1 was called with state', state, 'and action', action)
  if (typeof state === 'undefined') {
      return {}
  }

  return state;
}

var store_1 = createStore(reducer_1)
Output: reducer_1 was called with state undefined and action { type: '@@redux/INIT' }
```
console.log('store_1 state after initialization:', store_1.getState())
Output: store_1 state after initialization: {}

As expected, the state returned by Redux after initialization is now {}

There is however a much cleaner way to implement this pattern thanks to ES6:

var reducer_2 = function (state = {}, action) {
  console.log('reducer_2 was called with state', state, 'and action', action)

  return state;
}

var store_2 = createStore(reducer_2)
Output: reducer_2 was called with state {} and action { type: '@@redux/INIT' }

console.log('store_2 state after initialization:', store_2.getState())
Output: store_2 state after initialization: {}

You've probably noticed that since we've used the default parameter on state parameter of reducer_2,
we no longer get undefined as state's value in our reducer's body.

Let's now recall that a reducer is only called in response to an action dispatched and
let's fake a state modification in response to an action type 'SAY_SOMETHING'

var reducer_3 = function (state = {}, action) {
  console.log('reducer_3 was called with state', state, 'and action', action)

  switch (action.type) {
      case 'SAY_SOMETHING':
          return {
              ...state,
              message: action.value
          }
      default:
          return state;
  }
}

var store_3 = createStore(reducer_3)
Output: reducer_3 was called with state {} and action { type: '@@redux/INIT' }

console.log('store_3 state after initialization:', store_3.getState())
Output: redux state after initialization: {}

Nothing new in our state so far since we did not dispatch any action yet. But there are few
important things to pay attention to in the last example:
   0) I assumed that our action contains a type and a value property. The type property is mostly
      a convention in flux actions and the value property could have been anything else.
   1) You'll often see the pattern involving a switch to respond appropriately
      to an action received in your reducers
   2) When using a switch, NEVER forget to have a "default: return state" because
      if you don't, you'll end up having your reducer return undefined (and lose your state).
   3) Notice how we returned a new state made by merging current state with { message: action.value },
      all that thanks to this awesome ES7 notation (Object Spread): { ...state, message: action.value }
   4) Note also that this ES7 Object Spread notation suits our example because it's doing a shallow
      copy of { message: action.value } over our state (meaning that first level properties of state
      are completely overwritten - as opposed to gracefully merged - by first level property of
      { message: action.value }). But if we had a more complex / nested data structure, you might choose
      to handle your state's updates very differently:
      - using Immutable.js (https:facebook.github.io/immutable-js/)
      - using Object.assign (https:developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
      - using manual merge
      - or whatever other strategy that suits your needs and the structure of your state since
        Redux is absolutely NOT opinionated on this (remember, Redux is a state container).

Now that we're starting to handle actions in our reducer let's talk about having multiple reducers and
combining them.

## 5.combine-reducers

我们现在来看一下什么是 reducer

var reducer_0 = function (state = {}, action) {
  console.log('reducer_0 was called with state', state, 'and action', action)

  switch (action.type) {
      case 'SAY_SOMETHING':
          return {
              ...state,
              message: action.value
          }
      default:
          return state;
  }
}

在继续之前，我们先来想象一下拥有很多 action 的 reducer 长什么样子

var reducer_1 = function (state = {}, action) {
  console.log('reducer_1 was called with state', state, 'and action', action)

  switch (action.type) {
      case 'SAY_SOMETHING':
          return {
              ...state,
              message: action.value
          }
      case 'DO_SOMETHING':
           ...
      case 'LEARN_SOMETHING':
           ...
      case 'HEAR_SOMETHING':
           ...
      case 'GO_SOMEWHERE':
           ...
       etc.
      default:
          return state;
  }
}

很显然，只有一个 reducer 是hold不住我们整个应用中所有 action 操作的（好吧，事实上它是 hold 得住的，
但这会变得很难维护。）

幸运的是，Redux 不关心我们到底是只有一个 reducer ，还是有一打（12个）reducer 。
如果我们有多个 reducer ，Redux 能帮我们合并成一个。

让我们来定义 2 个 reducer

var userReducer = function (state = {}, action) {
  console.log('userReducer was called with state', state, 'and action', action)

  switch (action.type) {
       etc.
      default:
          return state;
  }
}
var itemsReducer = function (state = [], action) {
  console.log('itemsReducer was called with state', state, 'and action', action)

  switch (action.type) {
       etc.
      default:
          return state;
  }
}

我希望你特别留意赋给每个 reducer 的初始 state ：
   1. 赋给 userReducer 的初始 state 是一个空对象，即 {}
   2. 赋给 itemsReducer 的初始 state 是一个空数组，即 []
赋予不同类型的值是为了说明 reducer 是可以处理任何类型的数据结构的。你完全可以选择那些符合你的需求的
数据结构作为 state 的值。（例如，字面量对象、数组、布尔值、字符串或其它不可变结构）

在这种多个 reducer 的模式下，我们可以让每个 reducer 只处理整个应用的部分 state 。

但我们需要知道，createStore 只接收一个 reducer 函数。

那么，我们怎么合并所有的 reducer ？ 我们又该如何告诉 Redux 每个 reducer 只处理一部分 state 呢？
其实这很简单。我们使用 combineReducers 辅助函数。

combineReducers 接收一个对象（hash）并返回一个函数，当 combineReducers 被调用时，它会去调用每个
reducer ，并把返回的每一块 state 重新组合成一个大 state 对象（也就是 Redux 中的 Store）。
长话短说，下面演示一下如何使用多个 reducer 创建一个 Redux 实例：

import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
  user: userReducer,
  items: itemsReducer
})
输出：
userReducer was called with state {} and action { type: '@@redux/INIT' }
userReducer was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_9.r.k.r.i.c.n.m.i' }
itemsReducer was called with state [] and action { type: '@@redux/INIT' }
itemsReducer was called with state [] and action { type: '@@redux/PROBE_UNKNOWN_ACTION_4.f.i.z.l.3.7.s.y.v.i' }
var store_0 = createStore(reducer)
输出：
userReducer was called with state {} and action { type: '@@redux/INIT' }
itemsReducer was called with state [] and action { type: '@@redux/INIT' }

正如你从输出中看到的，每个 reducer 都被正确地调用了（但接收了个 init action @@redux/INIT ）。
这个 action 是什么鬼？这是 combineReducers 实施的一次安全检查，用以确保 reducer 永远不会返回
undefined。请注意，在 combineReducers 中第一次调用 init action 时，其实是随机 action 来的，
但它们有个共同的目的 (即是做一个安全检查)。

console.log('store_0 state after initialization:', store_0.getState())
输出：
store_0 state after initialization: { user: {}, items: [] }

有趣的是，我们发现 Redux 正确处理了 state 的各个部分。最终的 state 完全是一个简单的对象，由
userReducer 和 itemsReducer 返回的部分 state 共同组成。
{
   user: {},  {} is the slice returned by our userReducer
   items: []  [] is the slice returned by our itemsReducer
}

由于我们为每个 reducer 初始化了一个特殊的值（userReducer 的是空对象 {} ，itemsReducer 的是空数
组 [] ）,所以在最终 Redux 的 state 中找到那些值并不是巧合。

现在，关于 reducer 如何工作我们已经有了清楚的理解。是时候去看看当 action 被分发（dispatch）时会对
Redux 的 state 有什么影响。

## 6.dispatch-action

迄今为止我们的关注点都是绑定我们的 reducer(s)，但我们还未 dispatch 任何一个 action。
我们将会用到上一章的 reducer ，并用它们处理一些 action：

var userReducer = function (state = {}, action) {
  console.log('userReducer was called with state', state, 'and action', action)

  switch (action.type) {
      case 'SET_NAME':
          return {
              ...state,
              name: action.name
          }
      default:
          return state;
  }
}
var itemsReducer = function (state = [], action) {
  console.log('itemsReducer was called with state', state, 'and action', action)

  switch (action.type) {
      case 'ADD_ITEM':
          return [
              ...state,
              action.item
          ]
      default:
          return state;
  }
}

import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
  user: userReducer,
  items: itemsReducer
})
var store_0 = createStore(reducer)


console.log("\n", '### It starts here')
console.log('store_0 state after initialization:', store_0.getState())
输出：
store_0 state after initialization: { user: {}, items: [] }

让我们来 dispatch 我们的第一个 action... 记住在 'simple-action-creator.js' 中所提到的：
   "为了 dispatch 一个 action，我们需要... 一个 dispatch 函数。" Captain obvious

我们所看到的 dispatch 函数，是 Redux 提供的，并且它会将 action 传递
给任何一个 reducer！disparch 函数本质上是 Redux
的实例属性 "dispatch"

简单地调用一下，去 dispatch 一个 action：

store_0.dispatch({
  type: 'AN_ACTION'
})
输出：
userReducer was called with state {} and action { type: 'AN_ACTION' }
itemsReducer was called with state [] and action { type: 'AN_ACTION' }

每一个 reducer 都有效地被调用了，但是没有一个 action type 是 reducer 需要的，
因此 state 是不会发生变化的：

console.log('store_0 state after action AN_ACTION:', store_0.getState())
输出：store_0 state after action AN_ACTION: { user: {}, items: [] }

但是，等一下！我们是不是可以用一个 action creator 去发送一个 action？我们确实可以
用一个 actionCreator，但由于它只是返回一个 action，那么就意味着它不会携带任何东西
到这个例子中。但为了面对未来遇到的困难，我们还是以正确的方式，
即以 flux 理论去做吧。让我们使用这个 action creator 发送一个我们想要的 action：

var setNameActionCreator = function (name) {
  return {
      type: 'SET_NAME',
      name: name
  }
}

store_0.dispatch(setNameActionCreator('bob'))
输出：
userReducer was called with state {} and action { type: 'SET_NAME', name: 'bob' }
itemsReducer was called with state [] and action { type: 'SET_NAME', name: 'bob' }

console.log('store_0 state after action SET_NAME:', store_0.getState())
输出：
store_0 state after action SET_NAME: { user: { name: 'bob' }, items: [] }

在应用中，我们已经实现了第一个 action 并且用它改变了 state！

但是这似乎太简单了，并且还不足以充当一个真实的用例。例如，
如果我们要在 dispatch action 之前做一些异步的操作，那应该怎么做呢？
我们将在下一章节 "dispatch-async-action.js" 中讨论这个问题

迄今为止，这些就是我们应用中的流程
ActionCreator -> Action -> dispatcher -> reducer

## 7.dispatch-async-action-1.js

在上节教程中我们知道了如何分发 action 以及这些 action 如何通过 reducer 函数修改应用状态。

但是，到目前为止，我们只考虑了一种情况，同步 action，准确地说是同步 action creator，它同步地创建 action，
也就是当 action creator 被调用时，action 会被立即返回。

我们来设想一个简单的异步场景：
1）用户点击『Say Hi in 2 seconds』按钮
2）当用户点击按钮『A』，我们希望经过两秒，视图显示一条消息『Hi』
3）两秒过去之后，更新视图，显示消息『Hi』

当然这条消息是应用的状态之一，所以我们必然将其存储于 Redux store。
但是我们希望的结果是，在调用 action creator 的两秒之后才把消息存入 store（因为如果立即更新状态，
那么就会立即触发所有监听状态变更的订阅者 —— 例如视图，导致消息早于两秒显示）。

如果我们按照目前调用 action creator 的方式...

import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
  speaker: function (state = {}, action) {
      console.log('speaker was called with state', state, 'and action', action)

      switch (action.type) {
          case 'SAY':
              return {
                  ...state,
                  message: action.message
              }
          default:
              return state;
      }
  }
})
var store_0 = createStore(reducer)

var sayActionCreator = function (message) {
  return {
      type: 'SAY',
      message
  }
}

console.log("\n", 'Running our normal action creator:', "\n")

console.log(new Date());
store_0.dispatch(sayActionCreator('Hi'))

console.log(new Date());
console.log('store_0 state after action SAY:', store_0.getState())

输出（忽略初始输出）：
   Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
   speaker was called with state {} and action { type: 'SAY', message: 'Hi' }
   Sun Aug 02 2015 01:03:05 GMT+0200 (CEST)
   store_0 state after action SAY: { speaker: { message: 'Hi' } }


... 结果 store 被立即更新了。

我们希望看到的结果应该类似于下面这样的代码：

var asyncSayActionCreator_0 = function (message) {
  setTimeout(function () {
      return {
          type: 'SAY',
          message
      }
  }, 2000)
}

但是这样 action creator 返回的不是 action 而是 undefined。所以这并不是我们所期望的解决方法。

这里有个诀窍：不返回 action，而是返回 function。这个 function 会在合适的时机分发 action。但是如果我们希望
这个 function 能够分发 action，那么就需要向它传入 dispatch 函数。于是代码类似如下：

var asyncSayActionCreator_1 = function (message) {
  return function (dispatch) {
      setTimeout(function () {
          dispatch({
              type: 'SAY',
              message
          })
      }, 2000)
  }
}

你可能再次注意到 action creator 返回的不是 action 而是 function。
所以 reducer 函数很可能不知道如何处理这样的返回值，而你也并不清楚是否可行，那么让我们一起再做尝试，一探究竟。

## 8.dispatch-async-action-2

运行之前我们在 dispatch-async-action-1.js 中实现的第一个异步 action creator：

import { createStore, combineReducers } from 'redux'

var reducer = combineReducers({
  speaker: function (state = {}, action) {
      console.log('speaker was called with state', state, 'and action', action)

      switch (action.type) {
          case 'SAY':
              return {
                  ...state,
                  message: action.message
              }
          default:
              return state;
      }
  }
})
var store_0 = createStore(reducer)

var asyncSayActionCreator_1 = function (message) {
  return function (dispatch) {
      setTimeout(function () {
          dispatch({
              type: 'SAY',
              message
          })
      }, 2000)
  }
}

console.log("\n", 'Running our async action creator:', "\n")
store_0.dispatch(asyncSayActionCreator_1('Hi'))

输出：
   ...
   /Users/classtar/Codes/redux-tutorial/node_modules/redux/node_modules/invariant/invariant.js:51
       throw error;
             ^
   Error: Invariant Violation: Actions must be plain objects. Use custom middleware for async actions.
   ...

我们所设计的 function 似乎没有进入 reducer 函数。但是 Redux 给出了温馨提示：自定义中间件（middleware）实现异步 action。
看来我们的方向是正确的，可中间件（middleware）又是什么呢？

我向你保证 action creator asyncSayActionCreator_1 不仅没有问题，而且只要我们搞清楚 middleware 的概念并掌握它的使用方法，
这个异步 action creator 就会按照我们所设想的结果执行的。

## 9.middleware

We left dispatch-async-action-2.js with a new concept: "middleware". Somehow middleware should help us
to solve async action handling. So what exactly is middleware?

Generally speaking middleware is something that goes between parts A and B of an application to
transform what A sends before passing it to B. So instead of having:
A -----> B
we end up having
A ---> middleware 1 ---> middleware 2 ---> middleware 3 --> ... ---> B

How could middleware help us in the Redux context? Well it seems that the function that we are
returning from our async action creator cannot be handled natively by Redux but if we had a
middleware between our action creator and our reducers, we could transform this function into something
that suits Redux:

action ---> dispatcher ---> middleware 1 ---> middleware 2 ---> reducers

Our middleware will be called each time an action (or whatever else, like a function in our
async action creator case) is dispatched and it should be able to help our action creator
dispatch the real action when it wants to (or do nothing - this is a totally valid and
sometimes desired behavior).

In Redux, middleware are functions that must conform to a very specific signature and follow
a strict structure:
/*
  var anyMiddleware = function ({ dispatch, getState }) {
      return function(next) {
          return function (action) {
               your middleware-specific code goes here
          }
      }
  }
*/

As you can see above, a middleware is made of 3 nested functions (that will get called sequentially):
1) The first level provide the dispatch function and a getState function (if your
   middleware or your action creator needs to read data from state) to the 2 other levels
2) The second level provide the next function that will allow you to explicitly hand over
   your transformed input to the next middleware or to Redux (so that Redux can finally call all reducers).
3) the third level provides the action received from the previous middleware or from your dispatch
   and can either trigger the next middleware (to let the action continue to flow) or process
   the action in any appropriate way.

Those of you who are trained to functional programming may have recognized above an opportunity
to apply a functional pattern: currying (if you aren't, don't worry, skipping the next 10 lines
won't affect your redux understanding). Using currying, you could simplify the above function like that:
/*
   "curry" may come any functional programming library (lodash, ramda, etc.)
  var thunkMiddleware = curry(
      ({dispatch, getState}, next, action) => (
           your middleware-specific code goes here
      )
  );
*/

The middleware we have to build for our async action creator is called a thunk middleware and
its code is provided here: https:github.com/gaearon/redux-thunk.
Here is what it looks like (with function body translated to es5 for readability):

var thunkMiddleware = function ({ dispatch, getState }) {
   console.log('Enter thunkMiddleware');
  return function(next) {
       console.log('Function "next" provided:', next);
      return function (action) {
           console.log('Handling action:', action);
          return typeof action === 'function' ?
              action(dispatch, getState) :
              next(action)
      }
  }
}

To tell Redux that we have one or more middlewares, we must use one of Redux's
helper functions: applyMiddleware.

"applyMiddleware" takes all your middlewares as parameters and returns a function to be called
with Redux createStore. When this last function is invoked, it will produce "a higher-order
store that applies middleware to a store's dispatch".
(from https:github.com/rackt/redux/blob/v1.0.0-rc/src/utils/applyMiddleware.js)

Here is how you would integrate a middleware to your Redux store:

import { createStore, combineReducers, applyMiddleware } from 'redux'

const finalCreateStore = applyMiddleware(thunkMiddleware)(createStore)
For multiple middlewares, write: applyMiddleware(middleware1, middleware2, ...)(createStore)

var reducer = combineReducers({
  speaker: function (state = {}, action) {
      console.log('speaker was called with state', state, 'and action', action)

      switch (action.type) {
          case 'SAY':
              return {
                  ...state,
                  message: action.message
              }
          default:
              return state
      }
  }
})

const store_0 = finalCreateStore(reducer)
Output:
   speaker was called with state {} and action { type: '@@redux/INIT' }
   speaker was called with state {} and action { type: '@@redux/PROBE_UNKNOWN_ACTION_s.b.4.z.a.x.a.j.o.r' }
   speaker was called with state {} and action { type: '@@redux/INIT' }

Now that we have our middleware-ready store instance, let's try again to dispatch our async action:

var asyncSayActionCreator_1 = function (message) {
  return function (dispatch) {
      setTimeout(function () {
          console.log(new Date(), 'Dispatch action now:')
          dispatch({
              type: 'SAY',
              message
          })
      }, 2000)
  }
}

console.log("\n", new Date(), 'Running our async action creator:', "\n")

store_0.dispatch(asyncSayActionCreator_1('Hi'))
Output:
   Mon Aug 03 2015 00:01:20 GMT+0200 (CEST) Running our async action creator:
   Mon Aug 03 2015 00:01:22 GMT+0200 (CEST) 'Dispatch action now:'
   speaker was called with state {} and action { type: 'SAY', message: 'Hi' }

Our action is correctly dispatched 2 seconds after our call the async action creator!

Just for your curiosity, here is how a middleware to log all actions that are dispatched, would
look like:

function logMiddleware ({ dispatch, getState }) {
  return function(next) {
      return function (action) {
          console.log('logMiddleware action received:', action)
          return next(action)
      }
  }
}

Same below for a middleware to discard all actions that goes through (not very useful as is
but with a bit of more logic it could selectively discard a few actions while passing others
to next middleware or Redux):
function discardMiddleware ({ dispatch, getState }) {
  return function(next) {
      return function (action) {
          console.log('discardMiddleware action received:', action)
      }
  }
}

Try to modify finalCreateStore call above by using the logMiddleware and / or the discardMiddleware
and see what happens...
For example, using:
   const finalCreateStore = applyMiddleware(discardMiddleware, thunkMiddleware)(createStore)
should make your actions never reach your thunkMiddleware and even less your reducers.

See http:rackt.org/redux/docs/introduction/Ecosystem.html, section Middlewares, to
see other middleware examples.

Let's sum up what we've learned so far:
1) We know how to write actions and action creators
2) We know how to dispatch our actions
3) We know how to handle custom actions like asynchronous actions thanks to middlewares

The only missing piece to close the loop of Flux application is to be notified about
state updates to be able to react to them (by re-rendering our components for example).

So how do we subscribe to our Redux store updates?

## 10.state-subscriber

We're close to having a complete Flux loop but we still miss one critical part:

_________      _________       ___________
|         |    | Change  |     |   React   |
|  Store  |----▶ events  |-----▶   Views   |
|_________|    |_________|     |___________|

Without it, we cannot update our views when the store changes.

Fortunately, there is a very simple way to "watch" over our Redux's store updates:

/*
  store.subscribe(function() {
       retrieve latest store state here
       Ex:
      console.log(store.getState());
  })
*/

Yeah... So simple that it almost makes us believe in Santa Claus again.

Let's try this out:

import { createStore, combineReducers } from 'redux'

var itemsReducer = function (state = [], action) {
  console.log('itemsReducer was called with state', state, 'and action', action)

  switch (action.type) {
      case 'ADD_ITEM':
          return [
              ...state,
              action.item
          ]
      default:
          return state;
  }
}

var reducer = combineReducers({ items: itemsReducer })
var store_0 = createStore(reducer)

store_0.subscribe(function() {
  console.log('store_0 has been updated. Latest store state:', store_0.getState());
   Update your views here
})

var addItemActionCreator = function (item) {
  return {
      type: 'ADD_ITEM',
      item: item
  }
}

store_0.dispatch(addItemActionCreator({ id: 1234, description: 'anything' }))

Output:
   ...
   store_0 has been updated. Latest store state: { items: [ { id: 1234, description: 'anything' } ] }

Our subscribe callback is correctly called and our store now contains the new item that we added.

Theoretically speaking we could stop here. Our Flux loop is closed, we understood all concepts that make
Flux and we saw that it is not that much of a mystery. But to be honest, there is still a lot to talk
about and a few things in the last example were intentionally left aside to keep the simplest form of this
last Flux concept:

- Our subscriber callback did not receive the state as a parameter, why?
- Since we did not receive our new state, we were bound to exploit our closured store (store_0) so this
   solution is not acceptable in a real multi-modules application...
- How do we actually update our views?
- How do we unsubscribe from store updates?
- More generally speaking, how should we integrate Redux with React?

We're now entering a more "Redux inside React" specific domain.

It is very important to understand that Redux is by no means bound to React. It is really a
"Predictable state container for JavaScript apps" and you can use it in many ways, a React
application just being one of them.

In that perspective we would be a bit lost if it wasn't for react-redux (https:github.com/rackt/react-redux).
Previously integrated inside Redux (before 1.0.0), this repository holds all the bindings we need to simplify
our life when using Redux inside React.

Back to our "subscribe" case... Why exactly do we have this subscribe function that seems so simple but at
the same time also seems to not provide enough features?

Its simplicity is actually its power! Redux, with its current minimalist API (including "subscribe") is
highly extensible and this allows developers to build some crazy products like the Redux DevTools
(https:github.com/gaearon/redux-devtools).

But in the end we still need a "better" API to subscribe to our store changes. That's exactly what react-redux
brings us: an API that will allow us to seamlessly fill the gap between the raw Redux subscribing mechanism
and our developer expectations. In the end, you won't need to use "subscribe" directly. Instead you will
use bindings such as "provide" or "connect" and those will hide from you the "subscribe" method.

So yeah, the "subscribe" method will still be used but it will be done through a higher order API that
handles access to redux state for you.

We'll now cover those bindings and show how simple it is to wire your components to Redux's state.

## 11.Provider-and-connect（实战）

这其实是教程的最后一章，一起聊聊如何把 Redux 和 React 绑定在一起。

要运行下面的示例，你需要一个浏览器。

本示例中的代码和注释都在 ./11_src/src/ 目录下。

当你读到下面这段话的时间，请运行 11_src/src/server.js。

开发一个 React 应用和服务器来让浏览器可以访问，我们会用到：
- 用 node HTTP(https:nodejs.org/api/http.html) 创建一个非常简单的服务器
- 用 Webpack 去打包我们的应用，
- 神奇的 Webpack Dev Server (http:webpack.github.io/docs/webpack-dev-server.html)
 作为一个专门的 node 服务器，并监听 JS 改变自动编译
- 超棒的 React Hot Loader http:gaearon.github.io/react-hot-loader/ (Dan Abramov
 开发的另一个很棒的项目，没错，他就是 Redux 的作者) ，提供非常棒的
 DX (开发体验) ，当我们在编辑器中修改代码时，
 在浏览器中可以热加载来显示效果。

提醒一下 React 开发者：本应用是基于 React 0.14 构建的

我不想在这里详细地解释如何设置 Webpack Dev Server 和 React Hot Loader，
因为在 React Hot Loader 的文档中已经说的很好了。
import webpackDevServer from './11_src/src/webpack-dev-server'
我们应用启动的主要服务器请求都是来自这个文件。
import server from './11_src/src/server'

如果 5050 端口号已经被占用了，那么就修改下面的端口号。
如果端口号是 X，那么我们可以用 X 作为服务器的端口号，用 X+1 作为 webpack-dev-server 的端口号
const port = 5050

启动 webpack dev server...
webpackDevServer.listen(port)
... 还有主应用服务器。
server.listen(port)

console.log(`Server is listening on http:127.0.0.1:${port}`)

**转到 11_src/src/server.js...**

## 12.结语

There is actually more to Redux and react-redux than what we showed you with this tutorial. For example,
concerning Redux, you may be interested in bindActionCreators (to produce a hash of action creators
already bound to dispatch - http:rackt.org/redux/docs/api/bindActionCreators.html).

We hope we've given you the keys to better understand Flux and to see more clearly
how Flux implementations differ from one another--especially how Redux stands out ;).

Where to go from here?

The official Redux documentation is really awesome and exhaustive so you should not hesitate to
refer to it from now on: http:rackt.org/redux/index.html

Have fun with React and Redux!


## 参考资料

### Flux
- [Flux](https:github.com/react-guide/flux-cn)
- [Flux应用架构](http:reactjs.cn/react/docs/flux-overview.html)
- [flux docs](http:facebook.github.io/flux/docs/overview.html#content)
### Redux
- [Redux 中文文档](http:camsong.github.io/redux-in-chinese/index.html)
- [redux-tutorial-cn](https:github.com/react-guide/redux-tutorial-cn#redux-tutorial)
- [redux on github](https:github.com/reactjs/redux)
- [redux-tutorial](https:github.com/happypoulp/redux-tutorial)
