# 让Redux来管理你的应用

> [Redux 中文文档](http://camsong.github.io/redux-in-chinese/index.html)中的内容特别详细，本文部分内容参考自该文档。

在开始往下阅读之前，我默认你已经学习了前面的课程，并且掌握了Webpack、ES6、React等知识的应用。

在前面的课程，我们已经使用React创建了一个应用，但是在实际项目中，面对复杂业务逻辑的挑战，如何清晰高效的管理应用内的数据流动成为了关键。

Flux思想已经在提出后得到逐步推广，并广泛应用到实际的项目中。facebook的flux实现，开源社区的reflux、redux等类库开始涌现并得到了广大开发者的认同和使用。

Redux以其简单易用、文档齐全易懂等优点在开源社区得到开发者的一致好评，所以接下来让我们一起走进Redux，学习并将其使用到我们实际的项目开发中。

## 1. 重要的开始

React 已经帮我们在视图层解决了禁止异步和直接操作 DOM 等问题，让页面的高效渲染和组件化开发成为了可能。**美中不足的是，React 依旧把处理 state 中数据的问题留给了你，那么，Redux的出现就是为了帮你解决这个问题。**

### 1.1 对 Redux 的介绍

> - Redux 是 JavaScript 状态容器，提供可预测化的状态管理
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

### 1.3 快速上手

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

## 2. 理解 Redux 的核心概念
### 2.1 reducer

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

### 2.2 store

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

### 2.3 Redux 和 React Component的基本结合

使用react-dom进行简单的渲染
```
import { createStore } from 'redux';
import React from 'react';
import { render } from 'react-dom';

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

const Counter = ({value}) => (
  <h1>{value}</h1>
)

const PureRender = () => {
  render(
      <Counter value={store.getState()} />,
      document.getElementById('app')
  )
}

// store subscribe 订阅或是监听view（on）
store.subscribe(PureRender);
PureRender();

```

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

### 2.4 action

Action 是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

### 2.5 combineReducers

模拟这个方法
```
const combineReducers = (reducers) => {
  return (state = {}, action) => {
      return Object.keys(reducers).reduce(
        (nextState, key) => {
          nextState,[key] = reducers[key](
            state[key],
            action
          );
          return nextState;
        },{}
      )
  }
}
```

## 3. 使用 React-redux 连接 react 和 redux


### 3.1 一个最简单的完整示例

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

### 3.2 一步步开发一个 TODO 应用


## 4. 开发工具 Redux-devtools 的使用

## 5. Redux 的高级运用

## 6. 一些重要而实用的开发技巧


## 参考资料


- [Redux 中文文档](http:camsong.github.io/redux-in-chinese/index.html)
- [redux-tutorial-cn](https:github.com/react-guide/redux-tutorial-cn#redux-tutorial)
- [redux on github](https:github.com/reactjs/redux)
- [redux-tutorial](https:github.com/happypoulp/redux-tutorial)
- [Redux 核心概念](http://www.jianshu.com/p/3334467e4b32)
