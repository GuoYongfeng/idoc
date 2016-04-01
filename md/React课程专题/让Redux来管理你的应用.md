# 让Redux来管理你的应用

在开始往下阅读之前，我默认你已经学习了前面的课程，并且掌握了Webpack、ES6、React等知识的应用。

在前面的课程，我们已经使用React创建了一个应用，但是在实际项目中，复杂的业务逻辑，如何清晰高效的管理应用内的state成为关键。

Flux思想已经在这两年甚嚣尘上，facebook的flux实现，开源社区的reflux、redux等类库开始涌现并得到了广大开发者的认同和使用。

Redux以其简单易用、文档齐全易懂等优点在开源社区得到开发者的一致好评，所以接下来让我们一起走进Redux，学习并将其使用到我们实际的项目开发中。

## 0. 示例体验
- 下载[项目脚手架](https://github.com/GuoYongfeng/webpack-dev-boilerplate)，并将其跑通运行起来，这是我们的约定。
- 下载`redux`
```
$ npm install redux --save
```

暂不介绍Redux的概念和API，先上示例代码。
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

## 1. reducer

reducer是一个纯函数，传入state和action，返回一个新的state tree，简单而纯粹的完成某一件具体的事情，没有依赖，简约大气。

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

## 2. store

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

## 3. Redux 和 React Component的基本结合

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

## 4. action && action creator

## 5. Provider

## 6. combineReducers

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


## 参考资料


- [Redux 中文文档](http:camsong.github.io/redux-in-chinese/index.html)
- [redux-tutorial-cn](https:github.com/react-guide/redux-tutorial-cn#redux-tutorial)
- [redux on github](https:github.com/reactjs/redux)
- [redux-tutorial](https:github.com/happypoulp/redux-tutorial)
- [Redux 核心概念](http://www.jianshu.com/p/3334467e4b32)
