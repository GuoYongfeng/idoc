<h1 style="font-size: 40px;text-align:center;color: #007cdc;">深入理解Redux的Middleware</h1>

> 声明：本文基于kazaff翻译的[https://medium.com/@meagle/understanding-87566abcfb7a](https://medium.com/@meagle/understanding-87566abcfb7a)修改，内容仅供交流学习。

通过前面两部分的知识讲解，我们可以使用Redux进行项目开发了。但是，要用好Redux，我们需要深入理解并使用其Middleware，并了解其内部实现。我开始了Redux源码的探索，要想能更好的理解源码，你必须非常熟悉函数式编程理念。它的代码写的非常简练，没有扎实的基本功你会觉得非常难看懂。我将尝试带领你了解一些需要用到的函数式编程概念，包括：复合函数，柯里化，和高阶函数等。

我鼓励你去阅读源码来搞明白reducer函数是如何创建和组合的，action creator函数是如何作用于dispatch方法的，还有如何增加中间件来影响默认的dispatch方法的。这篇文章将会帮你理解一些Redux框架的源码，提升你的函数式编程能力，同时也让你尝试一下部分ES6语法。

## 1.中间件

Redux最有趣的一个概念是它允许你通过自定义的中间件来影响你store的dispatch逻辑。然而，当我第一次试图查看[applyMiddleware.js](http://rackt.github.io/redux/docs/api/applyMiddleware.html)源码时，可把本宝宝吓坏了。所以我只能尝试参照Redux中间件的[文档](http://rackt.github.io/redux/docs/api/applyMiddleware.html)从功能层面来分析代码。看了下面[这段话](http://rackt.github.io/redux/docs/api/applyMiddleware.html)，让我找回了面子：

> “中间件”这个词听起来很恐怖，但它实际一点都不难。想更好的了解中间件的方法就是看一下那些已经实现了的中间件是怎么工作的，然后尝试自己写一个。函数嵌套写法看起来很恐怖，但是大多数你能找到的中间件，代码都不超过十行，但是它们的强大来自于它们的可嵌套组合性。

区区10行的中间件很容易写，但是你要想明白它们是如何放入中间件调用链，又是如何影响store的diapatch方法的，还真需要一些经验。首先让我们来简单定义一下中间件到底是个啥，并且找一些简单的中间件看一下它们的具体实现方式。关于中间件的定义我能找到的最简单的描述是：

> 中间件主要被用于分离那些不属于你应用的核心业务逻辑的可被组合起来使用的代码。

听起来不算太复杂，对吧;)

如果你之前用过[Koa.js](http://koajs.com/)n，那你可能早就接触过中间件这个概念了。我最早使用中间件是在我用Java Servlet写Filters和用Ruby写类似认证授权，日志，性能监控或一些需要在业务逻辑执行前处理的模块。

Redux的中间件主要用在store的dispatch函数上。dispatch函数的作用是发送actions给一个或多个reducer来影响应用状态的。中间件可以增强默认的dispatch函数，我们来看一下Redux1.0.1版本的applyMiddleware源码：

	export default function applyMiddleware(...middlewares) {    		return (next)  =>

    		(reducer, initialState) => {

      			var store = next(reducer, initialState);
      			var dispatch = store.dispatch;
      			var chain = [];

      			var middlewareAPI = {
        			getState: store.getState,
        			dispatch: (action) => dispatch(action)
      			};

      			chain = middlewares.map(middleware =>
                    			middleware(middlewareAPI));

      			dispatch = compose(...chain, store.dispatch);
      			return {
        			...store,
        			dispatch
      			};
   			};
	}

就这么点儿代码，就使用了非常多的函数式编程的思想，包括：高阶函数，复合函数，柯里化和ES6语法。一开始我反复读了十遍，然后我疯了:)。让我们先来分别看一些函数式编程的知识，回头再理会上面这段让人不爽的代码。

## 2.函数式编程概念

在开始阅读Redux中间件源码之前，你需要先掌握一些函数式编程知识，如果你已经自学成才了请跳过本节。

### 2.1 复合函数

函数式编程是非常理论和非常数学化的。用数学的视角来解释复合函数，如下：

	given:
		f(x) = x^2 + 3x + 1
		g(x) = 2x

	then:
		(f ∘ g)(x) = f(g(x)) = f(2x) = 4x^2 + 6x + 1

你可以将上面的方式扩展到组合两个或更多个函数这都是可以的。我们再来看个例子，演示组合两个函数并返回一个新的函数：

	var greet = function(x) { return `Hello, ${ x }` };
	var emote = function(x) { return `${x} :)` };

	var compose = function(f, g) {
  		return function(x) {
    		return f(g(x));
  		}
	}

	var happyGreeting = compose(greet, emote);
	// happyGreeting(“Mark”) -> Hello, Mark :)

我们当然可以组合更多的函数，这里只是给大家简单阐述一下基础概念。更多的常用技巧我们我们可以在Redux的源码中看到。

### 2.2 柯里化

另一个屌炸天的函数式编程概念被称之为：柯里化。（译者注：这段翻译起来太难了，我放弃了，推荐你看[这里](http://www.zhangxinxu.com/wordpress/2013/02/js-currying/)来了解这个概念）

	var curriedAdd = function(a) {
    	return function(b) {
        	return a + b;
    	};
	};

	var addTen = curriedAdd(10);
	addTen(10); //20


通过柯里化来组合你的函数，你可以创建一个强大的数据处理管道。

## 3.Redux的Dispatch函数

Redux的Store有一个dispatch函数，它关注你的应用实现的业务逻辑。你可以用它指派actions到你定义的reducer函数，用以更新你的应用状态。Redux的reducer函数接受一个当前状态参数和一个action参数，并返回一个新的状态对象：

	reducer:: state -> action -> state

指派action很像发送消息，如果我们假设要从一个列表中删除某个元素，action结构一般如下：

	{type: types.DELETE_ITEM, id: 1}

store会指派这个action对象到它所拥有的所有reducer函数来影响应用的状态，然而只有关注删除逻辑的reducer会真的修改状态。在此期间没人会关注到底是谁修改了状态，花了多长时间，或者记录一下变更前后的状态数据镜像。这些非核心关注点都可以交给中间件来完成。

## 4.Redux Middleware

Redux中间件被设计成可组合的，会在dispatch方法之前调用的函数。让我们来创建一个简单的日志中间件，它会简单的输出dispatch前后的应用状态。Redux中间件的签名如下：

	middleware:: next -> action -> retVal

我们的logger中间件实现如下：

	export default function createLogger({ getState }) {
  		return (next) =>
    		(action) => {
      			const console = window.console;
      			const prevState = getState();
      			const returnValue = next(action);
      			const nextState = getState();
      			const actionType = String(action.type);
      			const message = `action ${actionType}`;

      			console.log(`%c prev state`, `color: #9E9E9E`, prevState);
      			console.log(`%c action`, `color: #03A9F4`, action);
      			console.log(`%c next state`, `color: #4CAF50`, nextState);
      			return returnValue;
    	};
	}

注意，我们的`createLogger`接受的`getState`方法是由`applyMiddleware.js`注入进来的。使用它可以在内部的闭包中得到应用的当前状态。最后我们返回调用`next`创建的函数作为结果。`next`方法用于维护中间件调用链和dispatch，它返回一个接受`action`对象的柯里化函数，接受的`action`对象可以在中间件中被修改，再传递给下一个被调用的中间件，最终dispatch会使用中间件修改后的action来执行。

更健壮的logger中间件实现可以看[这里](https://www.npmjs.com/package/redux-logger)，为了节省时间，我们的logger中间件实现非常的简陋。

我们来看看上面的logger中间件的业务流程：

1. 得到当前的应用状态；
2. 将action指派给下一个中间件；
3. 调用链下游的中间件全部被执行；
4. store中的匹配reducer被执行；
5. 此时得到新的应用状态。

我们这里来看一个拥有2个中间件组件的例子：

![](https://cdn-images-1.medium.com/max/800/1*jNHs4JtDn9r00bs4iNSwjA.png)

## 5.剖析applyMiddleware.js

现在咱们已经知道Redux中间件是个啥，并且也掌握了足够的函数式编程知识，那就让我们再次尝试阅读`applyMiddleware.js`的源码来探个究竟吧。这次感觉比较好理解了吧：

	export default function applyMiddleware(...middlewares) {    		return (next)  =>

    		(reducer, initialState) => {

      			var store = next(reducer, initialState);
      			var dispatch = store.dispatch;
      			var chain = [];

      			var middlewareAPI = {
        			getState: store.getState,
        			dispatch: (action) => dispatch(action)
      			};

      			chain = middlewares.map(middleware =>
                    			middleware(middlewareAPI));

      			dispatch = compose(...chain, store.dispatch);
      			return {
        			...store,
        			dispatch
      			};
   			};
	}

`applyMiddleware`可能应该起一个更好一点的名字，谁能告诉我这是为谁来“申请中间件”？我觉得可以这么叫：`applyMiddlewareToStore`，这样是不是更加明确一些？

我们来一行一行分析代码，首先我们看方法签名：

	export default function applyMiddleware(...middlewares)

注意这里有个很有趣的写法，参数：`...middlewares`，这么定义允许我们调用时传入任意个数的中间件函数作为参数。接下来函数将返回一个接受`next`作为参数的函数：

	return (next) => (reducer, initialState) => {...}

`next`参数是一个被用来创建store的函数，你可以看一下[createStore.js](https://github.com/rackt/redux/blob/master/src/createStore.js)源码的实现细节。最后这个函数返回一个类似`createStore`的函数，不同的是它包含一个由中间件加工过的dispatch实现。

接下来我们通过调用`next`拿到store对象（译者注："Next we assign the store implementation to the function responsible for creating the new store (again see createStore.js). "这句实在翻译不来～）。我们用一个变量保存原始的dispatch函数，最后我们声明一个数组来存储我们创建的中间件链：

	var store = next(reducer, initialState);
	var dispatch = store.dispatch;
	var chain = [];

接下来的代码将`getState`和调用原始的`dispatch`函数注入给所有的中间件：

	var middlewareAPI = {
  		getState: store.getState,
  		dispatch: (action) => dispatch(action)
	};

	chain = middlewares.map(middleware =>
                    middleware(middlewareAPI));

然后我们根据中间件链创建一个加工过的dispatch实现：

	dispatch = compose(...chain, store.dispatch);

最tm精妙的地方就是上面这行，Redux提供的`compose`工具函数组合了我们的中间件链，`compose`实现如下：

	export default function compose(...funcs) {
 		return funcs.reduceRight((composed, f) => f(composed));
	}

碉堡了！上面的代码展示了中间件调用链是如何创建出来的。中间件调用链的顺序很重要，调用链类似下面这样：

	middlewareI(middlewareJ(middlewareK(store.dispatch)))(action)

现在知道为啥我们要掌握复合函数和柯里化概念了吧？最后我们只需要将新的store和调整过的dispatch函数返回即可：

	return {
 		...store,
 		dispatch
	};

上面这种写法意思是返回一个对象，该对象拥有store的所有属性，并增加一个dispatch函数属性，store里自带的那个原始dispatch函数会被覆盖。这种写法会被[Babel](https://babeljs.io/repl/)转化成：

	return _extends({}, store, { dispatch: _dispatch });

现在让我们将我们的logger中间件注入到dispatch中：

	import { createStore, applyMiddleware } from ‘redux’;
	import loggerMiddleware from ‘logger’;
	import rootReducer from ‘../reducers’;

	const createStoreWithMiddleware =
  		applyMiddleware(loggerMiddleware)(createStore);

	export default function configureStore(initialState) {
  		return createStoreWithMiddleware(rootReducer, initialState);
	}

	const store = configureStore();

## 6.异步中间件

我们已经会写基础的中间件了，我们就要玩儿点高深得了，整个能处理异步action的中间件咋样？让我们来看一下[redux-thunk](https://github.com/gaearon/redux-thunk)的更多细节。我们假设有一个包含异步请求的action，如下：

	function fetchQuote(symbol) {
   		requestQuote(symbol);
   		return fetch(`http://www.google.com/finance/info?q=${symbol}`)
      			.then(req => req.json())
      			.then(json => showCurrentQuote(symbol, json));
	}

上面代码并没有明显的调用dispatch来分派一个返回promise的action，我们需要使用redux-thunk中间件来延迟dipatch的执行：

	function fetchQuote(symbol) {
  		return dispatch => {
    		dispatch(requestQuote(symbol));
    		return fetch(`http://www.google.com/finance/info?q=${symbol}`)
      				.then(req => req.json())
      				.then(json => dispatch(showCurrentQuote(symbol, json)));
  		}
	}

注意这里的`dipatch`
和`getState`是由`applyMiddleware`函数注入进来的。现在我们就可以分派最终得到的action对象到store的reducers了。下面是类似redux-thunk的实现：

	export default function thunkMiddleware({ dispatch, getState }) {
  		return next =>
     			action =>
       				typeof action === ‘function’ ?
         				action(dispatch, getState) :
         				next(action);
	}

这个和你之前看到的中间件没什么太大不同。如果得到的action是个函数，就用`dispatch`和`getState`当作参数来调用它，否则就直接分派给store。你可以看一下Redux提供的更详细的[异步示例](https://github.com/rackt/redux/tree/master/examples)。另外还有一个支持promises的中间件是[redux-promise](https://github.com/acdlite/redux-promise)。我觉得选择使用哪个中间件可以根据性能来考量。

## 7.使用middleware实现异步 action 和异步数据流

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


### 7.1 入口
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

### 7.2 容器型组件

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


### 7.3 展示型组件

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


### 7.4 Action & Action Creator


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

### 7.4 reducers

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

### 7.5 store

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

## 更多内容推荐

如果你还是对函数式编程不太习惯，我鼓励你看一下来自[Brian Lonsdorf](https://twitter.com/drboolean)的优秀文献：

- [Hey Underscore, You are Doing it Wrong! ](http://www.youtube.com/watch?v=m3svKOdZijA)：它介绍了很多函数式编程知识和少量的Underscore内容；
- [Professor Frisby’s Mostly Adequate Guide to Functional Programming](http://drboolean.gitbooks.io/mostly-adequate-guide/content/index.html)

## 总结

希望你已经了解了关于Redux中间件的足够信息，我也希望你掌握了更多的关于函数式编程的知识。我不断的尝试更多更好的函数式编程方法，尽管一开始并不容易，你需要不断的学习和尝试来参悟它的精髓。如果你完全掌握了这篇文章交给你的，那么你已经拥有了足够的信心去投入更多的学习当中。

最后，千万别使用那些你还没有搞明白的第三方类库，你必须确定它会给你的项目带来好处。掌握它的一个好方法就是去阅读它的源码，你将会学到新的编程技术，淘汰那些老的解决方案。将一个工具引入你的项目前，你有责任搞清楚它的细节。
