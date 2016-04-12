<h1 style="font-size: 40px;text-align:center;color: blue;">让Redux来管理你的应用（二）</h1>

> **重要提示：** 本教程配套示例代码请前往[redux-complete-sample](https://github.com/GuoYongfeng/redux-complete-sample)下载，课程中会有大量的示例操作，操作说明均基于这个配套的示例代码仓库，所以为了方便学习，请务必下载安装并启动。


在前一部分内容[](http://guoyongfeng.github.io/idoc/html/React%E8%AF%BE%E7%A8%8B%E4%B8%93%E9%A2%98/%E8%AE%A9Redux%E6%9D%A5%E7%AE%A1%E7%90%86%E4%BD%A0%E7%9A%84%E5%BA%94%E7%94%A8%EF%BC%88%E4%B8%80%EF%BC%89.html)中，我们学习了基础部分的Redux知识，并且完成了几个示例练习。如果你已经都能够轻松掌握了，恭喜你，现在是时候来点好玩的干货了。

而且有计划地，我把上一部分可能涉及到相对不太好理解的内容都放到这里，希望我们一起来进入redux学习的深水区。

## 1.超酷的开发工具 Redux-devtools

redux-devtools是一个有趣而又高效的redux开发工具，如果你想直接在github上查看相关的内容，请前往[这里](https://github.com/gaearon/redux-devtools)。事实上，也鼓励大家养成在github上学习第一手资料的习惯。


ok，首先，让我们来下载redux-devtools的相关资料

```
$ npm install --save-dev redux-devtools redux-devtools-log-monitor redux-devtools-dock-monitor
```

- `redux-devtools：`redux的开发工具包，而且DevTools支持自定义的 monitor 组件，所以我们可以完全自定义一个我们想要的 monitor 组件的UI展示风格，以下两个是我们示例中用到的。
- `redux-devtools-log-monitor：` 这是Redux Devtools 默认的 monitor ，它可以展示state 和 action 的一系列信息，而且我们还可以在monitor改变它们的值。
- `redux-devtools-dock-monitor：`这monitor支持键盘的快捷键来轻松的改变tree view在浏览器中的展示位置，比如不断的按‘ctrl + q’组合键可以让展示工具停靠在浏览器的上下左右不同的位置，按“ctrl+h”组合键则可以控制展示工具在浏览器的显示或隐藏的状态。

接下来我们在containers目录新增一个DevTools.js文件，并且设置monitor。
代码清单：`demo-redux-devtools/containers/DevTools.js`
```
import React from 'react';
import { createDevTools } from 'redux-devtools';
import LogMonitor from 'redux-devtools-log-monitor';
import DockMonitor from 'redux-devtools-dock-monitor';

export default createDevTools(
  <DockMonitor toggleVisibilityKey='ctrl-h'
               changePositionKey='ctrl-q'>
    <LogMonitor />
  </DockMonitor>
);

```

然后在index.js文件引入这个容器组件
代码清单：`demo-redux-devtools/index.js`
```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import App from './containers/App'
import DevTools from './containers/DevTools'
import { createStore, compose } from 'redux';
import todoApp from './reducers'

// 把多个 store 增强器从右到左来组合起来，依次执行
// 这个地方完全可以不用compose，演示一下compose的使用
const enhancer = compose(
  DevTools.instrument()
);

let store = createStore(todoApp, enhancer)
let rootElement = document.getElementById('app')

render(
  <Provider store={store}>
    <div>
      <App />
      <DevTools />
    </div>
  </Provider>,
  rootElement
)

```

> **注意：**在实际项目开发中时应该根据环境来确定`<DevTools />`组件的展示与否，示例中是在开发环境的演示，直接放在`Provider`里面。

ok，重新启动示例应用即可，样子是这样的：

<img src="/img/redux/devtools.png" />

## 2.理解和使用 Middleware

如果你使用过 Express 或者 Koa 等服务端框架, 那么应该对 middleware 的概念不会陌生。 在这类框架中，middleware 是指可以被嵌入在框架接收请求到产生响应过程之中的代码。例如，Express 或者 Koa 的 middleware 可以完成添加 CORS headers、记录日志、内容压缩等工作。

middleware 最优秀的特性就是可以被链式组合。你可以在一个项目中使用多个独立的第三方 middleware。

相对于 Express 或者 Koa 的 middleware，Redux middleware 被用于解决不同的问题，但其中的概念是类似的。它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。**说了这么多，其实Redux中的middleware就是接受不同类型的action对象作为输入，负责改变store中的dispatch方法，从而达到我们预期的需求**。

我们可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

比如这样；
```
let next = store.dispatch

store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

> 这里封装的dispatchAndLog方法就是一个middleware，它接受一个action对象，然后在中间做了一些Log信息的打印工作，没有改变store，但是完成了打印日志的一个需求，而把这个功能完善并且独立成一个包，这就是接下来我们将要学习的middleware。

写一个示例测试一下：
```
$ cd demo-middleware && mkdir middleware
$ cd middleware && touch index.js
```

代码清单：`demo-middleware/middleware/index.js`
```
/**
 * 记录所有被发起的 action 以及产生的新的 state。
 */
export const logger = store => next => action => {
  console.group(action.type)
  console.info('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  console.groupEnd(action.type)
  return result
}

/**
 * 在 state 更新完成和 listener 被通知之后发送崩溃报告。
 */
export const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}

```

代码清单：`demo-middleware/index.js`
```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import App from './containers/App'
import { createStore, applyMiddleware, compose } from 'redux';
import todoApp from './reducers'
import { logger, crashReporter } from './middleware/index'

const newCreateStore = (initState) => {
  // 返回一个创建好的 store
  return createStore(
    todoApp,
    initState,
    // store enhancer
    applyMiddleware(logger, crashReporter)
  )
}

const store = newCreateStore();

let rootElement = document.getElementById('app')

render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)

```

- 这里使用redux的applyMiddleware方法将middlewares串起来，并返回一个增强型的 createStore 。
- compose 将从右到左来传入的多个函数组合起来

## 3.使用middleware实现异步 action 和异步数据流

redux的生态在持续的完善，其中就有不少优秀的 middleware 供开发者使用，同时大家也可以实现自己的middleware。

现在让我们来使用以下两个中间件来完成一个示例：
- [redux-thunk]() — Redux-Thunk可以让你的action creator返回一个function而不是action。这可以用于延迟dispath一个action或是在特定条件下dispatch才触发。他的内部函数接受store的dispath和getState方法作为参数。
- [redux-logger]() — redux-logger的用处很明显，就是用于记录所有 action 和下一次 state 的日志。

下面我们到示例项目的demo-middleware-apply下开始代码操作：

在此之前我们需要下载几个示例项目中依赖到的package
```
$ npm install --save-dev isomorphic-fetch redux-thunk redux-logger
```
- `isomorphic-fetch`：用于ajax请求数据


### 3.1 入口
`index.js`
```
import 'babel-polyfill'

import React from 'react'
import { render } from 'react-dom'
import Root from './containers/Root'

render(
  <Root />,
  document.getElementById('app')
)

```

### 3.2 容器型组件

`containers/Root.js`
```
import React, { Component } from 'react'
import { Provider } from 'react-redux'
import configureStore from '../store/configureStore'
import App from './App'

const store = configureStore()

export default class Root extends Component {
  render() {
    return (
      <Provider store={store}>
        <App />
      </Provider>
    )
  }
}

```

`containers/App.js`
```
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { selectSubreddit, fetchPostsIfNeeded, invalidateSubreddit } from '../actions'
import Picker from '../components/Picker'
import Posts from '../components/Posts'

class App extends Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
    this.handleRefreshClick = this.handleRefreshClick.bind(this)
  }

  componentDidMount() {
    const { dispatch, selectedSubreddit } = this.props
    dispatch(fetchPostsIfNeeded(selectedSubreddit))
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.selectedSubreddit !== this.props.selectedSubreddit) {
      const { dispatch, selectedSubreddit } = nextProps
      dispatch(fetchPostsIfNeeded(selectedSubreddit))
    }
  }

  handleChange(nextSubreddit) {
    this.props.dispatch(selectSubreddit(nextSubreddit))
  }

  handleRefreshClick(e) {
    e.preventDefault()

    const { dispatch, selectedSubreddit } = this.props
    dispatch(invalidateSubreddit(selectedSubreddit))
    dispatch(fetchPostsIfNeeded(selectedSubreddit))
  }

  render() {

    const { selectedSubreddit, posts, isFetching, lastUpdated } = this.props

    return (
      <div>
        <Picker value={selectedSubreddit}
                onChange={this.handleChange}
                options={[ 'guoyongfeng', 'tj' ]} />
        <p>
          {lastUpdated &&
            <span>
              Last updated at {new Date(lastUpdated).toLocaleTimeString()}.
              {' '}
            </span>
          }
          {!isFetching &&
            <a href='#'
               onClick={this.handleRefreshClick}>
              Refresh
            </a>
          }
        </p>
        {isFetching && posts.length === 0 &&
          <h2>请稍等...</h2>
        }
        {!isFetching && posts.length === 0 &&
          <h2>暂无数据.</h2>
        }
        <div style={{ opacity: isFetching ? 0.5 : 1 }}>
          <Posts posts={posts} />
        </div>

      </div>
    )
  }
}

App.propTypes = {
  selectedSubreddit: PropTypes.string.isRequired,
  posts: PropTypes.object.isRequired,
  isFetching: PropTypes.bool.isRequired,
  lastUpdated: PropTypes.number,
  dispatch: PropTypes.func.isRequired
}

function mapStateToProps(state) {
  const { selectedSubreddit, postsBySubreddit } = state
  const {
    isFetching,
    lastUpdated,
    items: posts
  } = postsBySubreddit[selectedSubreddit] || {
    isFetching: true,
    items: {}
  }

  return {
    selectedSubreddit,
    posts,
    isFetching,
    lastUpdated
  }
}

export default connect(mapStateToProps)(App)

```


### 3.3 展示型组件

`components/Picker.js`
```
import React, { Component, PropTypes } from 'react'

export default class Picker extends Component {
  render() {
    const { value, onChange, options } = this.props

    return (
      <span>
        <h1>{value}</h1>
        <select onChange={e => onChange(e.target.value)}
                value={value}>
          {options.map(option =>
            <option value={option} key={option}>
              {option}
            </option>)
          }
        </select>
      </span>
    )
  }
}

Picker.propTypes = {
  options: PropTypes.arrayOf(
    PropTypes.string.isRequired
  ).isRequired,
  value: PropTypes.string.isRequired,
  onChange: PropTypes.func.isRequired
}

```

`components/Posts.js`
```
import React, { PropTypes, Component } from 'react'

export default class Posts extends Component {
  render() {
    let bio = this.props.posts;

    return (
      <div>
        <h3> 用户信息 </h3>
          {bio.avatar_url && <li className="list-group-item"> <img src={bio.avatar_url} className="img-rounded img-responsive"/></li>}
          {bio.name && <li className="list-group-item">Name: {bio.name}</li>}
          {bio.login && <li className="list-group-item">Username: {bio.login}</li>}
          {bio.email && <li className="list-group-item">Email: {bio.email}</li>}
          {bio.location && <li className="list-group-item">Location: {bio.location}</li>}
          {bio.company && <li className="list-group-item">Company: {bio.company}</li>}
          {bio.followers && <li className="list-group-item">Followers: {bio.followers}</li>}
          {bio.following && <li className="list-group-item">Following: {bio.following}</li>}
          {bio.public_repos && <li className="list-group-item">Public Repos: {bio.public_repos}</li>}
          {bio.blog && <li className="list-group-item">Blog: <a href={bio.blog}> {bio.blog}</a></li>}
      </div>
    )
  }
}

Posts.propTypes = {
  posts: PropTypes.object.isRequired
}

```


### 3.4 Action & Action Creator


`actions/index.js`
```
import fetch from 'isomorphic-fetch'

export const REQUEST_POSTS = 'REQUEST_POSTS'
export const RECEIVE_POSTS = 'RECEIVE_POSTS'
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}

function receivePosts(subreddit, json) {

  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json,
    receivedAt: Date.now()
  }
}

function fetchPosts(subreddit) {

  return dispatch => {
    return fetch(`https://api.github.com/users/${subreddit}`)
      .then(response => response.json())
      .then(json => {

        dispatch(receivePosts(subreddit, json))
      })
  }
}

function shouldFetchPosts(state, subreddit) {
  const posts = state.postsBySubreddit[subreddit]
  if (!posts) {
    return true
  } else if (posts.isFetching) {
    return false
  } else {
    return posts.didInvalidate
  }
}

export function fetchPostsIfNeeded(subreddit) {
  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      return dispatch(fetchPosts(subreddit))
    }
  }
}

```

### 3.4 reducers

`reducers/index.js`
```
import { combineReducers } from 'redux'
import {
  SELECT_SUBREDDIT, INVALIDATE_SUBREDDIT,
  REQUEST_POSTS, RECEIVE_POSTS
} from '../actions'

function selectedSubreddit(state = 'guoyongfeng', action) {
  switch (action.type) {
  case SELECT_SUBREDDIT:
    return action.subreddit
  default:
    return state
  }
}

function posts(state = {
  isFetching: false,
  didInvalidate: false,
  items: {}
}, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
      return Object.assign({}, state, {
        didInvalidate: true
      })
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        isFetching: true,
        didInvalidate: false
      })
    case RECEIVE_POSTS:
      return Object.assign({}, state, {
        isFetching: false,
        didInvalidate: false,
        items: action.posts,
        lastUpdated: action.receivedAt
      })
    default:
      return state
  }
}

function postsBySubreddit(state = { }, action) {
  switch (action.type) {
    case INVALIDATE_SUBREDDIT:
    case RECEIVE_POSTS:
    case REQUEST_POSTS:
      return Object.assign({}, state, {
        [action.subreddit]: posts(state[action.subreddit], action)
      })
    default:
      return state
  }
}

const rootReducer = combineReducers({
  postsBySubreddit,
  selectedSubreddit
})

export default rootReducer

```

### 3.5 store

`store/configureStore.js`
```
import { createStore, applyMiddleware } from 'redux'
import thunkMiddleware from 'redux-thunk'
import createLogger from 'redux-logger'
import rootReducer from '../reducers'

const loggerMiddleware = createLogger()

export default function configureStore(initialState) {
  return createStore(
    rootReducer,
    initialState,
    applyMiddleware(
      thunkMiddleware,
      loggerMiddleware
    )
  )
}

```

最后，启动服务
```
$ cd demo-middleware-apply
$ webpack-dev-server --progress --colors
```

示例展示的效果是这样的

<img src="/img/redux/middleware.png" />


## 参考资料

- [Redux 中文文档](http://cn.redux.js.org/)
- [redux on github](https:github.com/reactjs/redux)
