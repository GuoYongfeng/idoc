<h1 style="font-size: 40px;text-align:center;color: blue;">让Redux来管理你的应用（一）</h1>

> [Redux 中文文档](http://camsong.github.io/redux-in-chinese/index.html)中的内容特别详细，本文部分内容参考自该文档。

在开始往下阅读之前，我默认你已经学习了前面的课程，并且掌握了Webpack、ES6、React等知识的应用。

在前面的课程，我们已经使用React创建了一个应用，但是在实际项目中，面对复杂业务逻辑的挑战，如何清晰高效的管理应用内的数据流动成为了关键。

Flux思想已经在提出后得到逐步推广，并广泛应用到实际的项目中。facebook的flux实现，开源社区的reflux、redux等类库开始涌现并得到了广大开发者的认同和使用。

Redux以其简单易用、文档齐全易懂等优点在开源社区得到开发者的一致好评，所以接下来让我们一起走进Redux，学习并将其使用到我们实际的项目开发中。

## 1. 基本介绍

React 已经帮我们在视图层解决了禁止异步和直接操作 DOM 等问题，让页面的高效渲染和组件化开发成为了可能。美中不足的是，React 依旧把处理 state 中数据的问题留给了你，那么，Redux的出现就是为了帮你解决这个问题。

### 1.1 对 Redux 的介绍

- Redux 是 JavaScript 状态容器，提供可预测化的状态管理
- 它可以让你构建一致化的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试
- 还提供 redux-devtools 让开发者享受超爽的开发体验
- 体小精悍（只有2kB）且没有任何依赖
- 拥有丰富的生态圈：教程、开发者工具、路由、组件、中间件、工具集...


### 1.2 Flux 和 Redux的对比

开始之前，我们不妨对比下flux的实现逻辑和redux的实现逻辑

<img src="/img/redux/flux.jpg" />

<p style="text-align:center;color:blue;">Flux实现原理</p>

<img src="/img/redux/redux.jpg" />

<p style="text-align:center;color:blue;">Redux实现原理</p>

## 2. 快速上手

我们约定，后续内容的操作练习都基于[项目脚手架](https://github.com/GuoYongfeng/webpack-dev-boilerplate)，首先将该项目下载并跑通运行。
```
$ git clone git@github.com:GuoYongfeng/webpack-dev-boilerplate.git my-redux-demo
$ cd my-redux-demo
$ npm install
$ npm run dev
```

下载安装 `redux`

```
$ npm install redux --save
```

示例代码快速体验：

```
/* app/index.js */
import { createStore } from 'redux';

// 这是一个 reducer，形式为 (state, action) => state 的纯函数。描述了 action 如何把 state 转变成下一个 state。

// state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter);

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() =>
  console.log(store.getState())
);

// 改变内部 state 惟一方法是 dispatch 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: 'INCREMENT' });
// 1
store.dispatch({ type: 'INCREMENT' });
// 2
store.dispatch({ type: 'DECREMENT' });
// 1
```

```
/* app/index.js */
import { createStore } from 'redux';

// reducer 纯函数，具体的action执行逻辑
const counter = (state = 0, action) => {
  switch (action.type) {
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
  }
}

// create store
const store = createStore(counter);

// view 对应到React里面的component
const PureRender = () => {
  document.body.innerText = store.getState();
}

// store subscribe 订阅或是监听view（on）
store.subscribe(PureRender);
PureRender();

document.addEventListener('click', function( e ){
  // store dispatch 调度分发一个 action（fire）
  store.dispatch({ type: 'DECREMENT'});
})

```

## 3. 理解 Redux 的核心概念

### 3.1 reducer

我们先来看一下 Javascript 中 Array.prototype.reduce 的用法：
```
const initState = '';
const actions = ['a', 'b', 'c'];
const newState = actions.reduce(
    ( (prevState, action) => prevState + action ),
    initState
);
```

对应的理解，Redux 中的 reducer 是一个纯函数，传入state和action，返回一个新的state tree，简单而纯粹的完成某一件具体的事情，没有依赖，**简单而纯粹是它的标签**。

```
const counter = (state = 0, action) => {
  switch (action.type) {
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
  }
}
```

### 3.2 store

```
/* app/index.js */
// reducer 纯函数，具体的action执行逻辑
const counter = (state = 0, action) => {
  switch (action.type) {
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
  }
}

// 模拟create store，了解其原理
const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  }

  const subscribe = (listener) => {
      listeners.push(listener);
      return () => {
        listeners = listeners.filter(item => item !== listener);
      }
  }

  dispatch({});

  return { getState, dispatch, subscribe };
}

const store = createStore(counter);

// view 对应到React里面的component
const PureRender = () => {
  document.body.innerText = store.getState();
}

// store subscribe 订阅或是监听view（on）
store.subscribe(PureRender);
PureRender();

document.addEventListener('click', function( e ){
  // store dispatch 调度分发一个 action（fire）
  store.dispatch({ type: 'DECREMENT'});
})

```

> - getState() : 返回应用当前的 state 树。
- dispatch(action) ： 分发 action。这是触发 state 变化的惟一途径。
- subscribe(listener) ： 添加一个变化监听器。每当 dispatch action 的时候就会执行，state 树中的一部分可能已经变化。你可以在回调函数里调用 getState() 来拿到当前 state。这是一个底层 API。多数情况下，你不会直接使用它，会使用一些 React（或其它库）的绑定

### 3.3 action

Action 是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

## 4. Redux 的顶层 API 介绍

### 4.1 createStore

调用方式：createStore(reducer, [initialState])


创建一个 Redux store 来以存放应用中所有的 state，应用中应有且仅有一个 store。
这个API返回一个store，这个store中保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。你也可以 subscribe 监听 state 的变化，然后更新 UI。我们来看一个示例

```
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([ action.text ])
    default:
      return state
  }
}

let store = createStore(todos, [ 'Use Redux' ])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

### 4.2 combineReducers

调用方式：combineReducers(reducers)

随着应用变得复杂，需要对 [reducer 函数](../Glossary.md#reducer) 进行拆分，拆分后的每一块独立负责管理 [state](../Glossary.md#state) 的一部分。

`combineReducers` 辅助函数的作用是，把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 [`createStore`](createStore.md)。

合并后的 reducer 可以调用各个子 reducer，并把它们的结果合并成一个 state 对象。**state 对象的结构由传入的多个 reducer 的 key 决定**。

最终，state 对象的结构会是这样的：

```
{
  reducer1: ...
  reducer2: ...
}
```

### 4.3 applyMiddleware

调用方式：applyMiddleware(...middlewares)

### 4.4 bindActionCreators

调用方式：bindActionCreators(actionCreators, dispatch)

### 4.5 compose

调用方式：compose(...functions)

## 5. 使用 React-redux 连接 react 和 redux

### 5.1 没有react-redux的写法

封装一个组件，将组件和Redux做基本的组合
```
import { createStore } from 'redux';
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

// reducer 纯函数，具体的action执行逻辑
const counter = (state = 0, action) => {
  switch (action.type) {
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
  }
}

const store = createStore(counter);

// Counter 组件
class Counter extends Component {
  render(){
    return (
      <div>
        <h1>{this.props.value}</h1>
        <button onClick={this.props.onIncrement}>点击加1</button>
        <button onClick={this.props.onDecrement}>点击减1</button>
      </div>
    )
  }
}

const PureRender = () => {
  ReactDOM.render(
      <Counter
        value={store.getState()}
        onIncrement={ () => store.dispatch({type: "INCREMENT"}) }
        onDecrement={ () => store.dispatch({type: "DECREMENT"}) }
      />, document.getElementById('app')
  );
}

// store subscribe 订阅或是监听view（on）
store.subscribe(PureRender)
PureRender()

```


### 5.2 使用react-redux的一个简单完整示例

```
import React, { Component, PropTypes } from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import { Provider, connect } from 'react-redux'

// React component
class Counter extends Component {
  render() {
    const { value, onIncreaseClick } = this.props
    return (
      <div>
        <span>{value}</span>
        <button onClick={onIncreaseClick}>Increase</button>
      </div>
    )
  }
}

Counter.propTypes = {
  value: PropTypes.number.isRequired,
  onIncreaseClick: PropTypes.func.isRequired
}

// Action
const increaseAction = { type: 'increase' }

// Reducer
function counter(state = { count: 0 }, action) {
  let count = state.count
  switch (action.type) {
    case 'increase':
      return { count: count + 1 }
    default:
      return state
  }
}

// Store
let store = createStore(counter)

// Map Redux state to component props
function mapStateToProps(state) {
  return {
    value: state.count
  }
}

// Map Redux actions to component props
function mapDispatchToProps(dispatch) {
  return {
    onIncreaseClick: () => dispatch(increaseAction)
  }
}

// Connected Component
let App = connect(
  mapStateToProps,
  mapDispatchToProps
)(Counter)

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)

```

## 5.3 react-redux的API介绍

- Provider
- connect
- mapStateToProps & mapDispatchToProps

### 5.4 一步步开发一个 TODO 应用

## 参考资料

- [Redux 中文文档](http://cn.redux.js.org/)
- [redux on github](https:github.com/reactjs/redux)
