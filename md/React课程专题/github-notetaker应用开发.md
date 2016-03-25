# github-notetaker应用开发

## 0.介绍 INTRODUCTION

我们计划用最近所学的技术知识来完成一个完整且实用的功能应用，这个应用具有以下功能：

- 通过搜索github上的用户名来检索该用户的基本信息
- 可以检索到该用户的github上的所有代码仓库信息
- 可以对该用户进行简单的评论
- 通过特定路由可以访问特定用户的信息

大概是长这个样子

<img src="/img/notetaker/demo.png" />

## 1.启动
首先我们把脚手架项目下载下来并且安装启动
```
$ git clone git@github.com:GuoYongfeng/webpack-dev-boilerplate.git github-notetaker-app
$ cd github-note-app && npm install
$ npm run dev
```

<img src="/img/comment/dev.png" />

查看浏览器效果

<img src="/img/comment/init.png" />

## 2.使用react-router跑通基本路由

下载react-router和history

```
$ npm install react-router history --save
```

简单说一下react-router和history都是用来干嘛的：
- react-router is a complete routing library for React. react-router是一个专为react提供的完整路由库。
- history is a JavaScript library that lets you easily manage session history anywhere JavaScript runs. history是一个js库，可以让您轻松地管理会话历史。

接下来我们开始写代码：

代码清单：`app/index.js`
```
import React from 'react';
import ReactDOM from 'react-dom';
// 注意：这里使用的是browserHistory，需要在webpack-dev-server启动的时候加上参数 --history-api-fallback
import { Router, browserHistory  } from 'react-router';
import routes from './routes/index.jsx';

let root = document.getElementById('app');
ReactDOM.render(
  <Router routes={routes} history={browserHistory} />, root
);

```

这里我们引入了react-router里面的Router，为了方便路由管理，我们新建一个管理路由的目录，并且引入路由配置，接下来在app目录下创建一个路由表。

代码清单：
```
$ cd app && mkdir routes
$ cd routes && touch index.jsx
```

配置路由，代码清单：`app/routes/index.jsx`
```
import React from 'react';
import { Route, IndexRoute } from 'react-router';
import { App, Home } from '../containers';

export default (
  <Route path="/" component={App}>
    <IndexRoute component={Home} />
  </Route>
)
```

这里从react-router引入了Route和IndexRoute，其中Route就是用来配置单个具体的路由，IndexRoute是用于在路由中展示默认的组件，而且一级路由中还可以嵌套二级路由。

从container中引入了App和Home这两个容器组件，当访问路由"/"的时候，渲染的是组件App和Home组成的页面。

接下来对App.jsx修改，代码清单：`app/containers/App/App.jsx`
```
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <div>
        <h1> content from App Component </h1>
        {this.props.children}
      </div>
    );
  }
}

export default App;

```

同时，新增一个容器组件Home。
```
$ cd app/containers && mkdir Home
$ cd Home && touch Home.jsx
```

代码清单：`app/containers/Home/Home.jsx`
```
import React, { Component } from 'react';

class Home extends Component {
  render() {
    return (
      <h2>content from Home Component</h2>
    );
  }
}

export default Home;

```

这里我们新增了组件，同时在containers下面的组件索引文件中进行更新。
代码清单：`app/containers/index.js`
```
'use strict';

export App from './App/App.jsx';
export Home from './Home/Home.jsx';
```

初步完成路由的管理，我们接下来在浏览器中查看效果

<img src="/img/notetaker/router1.png" />

## 3.新增头部搜索组件

在新增组件开始，有这么个打算----为了省点懒，也为了快速的写出好看的页面，我们可以把bootstrap用起来。

```
$ npm install bootstrap --save
```

然后在app/index.js中引入样式文件，代码清单：`app/index.js`
```
import React from 'react';
import ReactDOM from 'react-dom';
import { Router, browserHistory  } from 'react-router';
import routes from './routes/index.jsx';
import 'bootstrap/dist/css/bootstrap.css';

let root = document.getElementById('app');
ReactDOM.render(
  <Router routes={routes} history={browserHistory} />, root
);

```

按照我们前面的原型图，我们先把头部搜索表单写好，搜索部分的搜索框和按钮部分，我们可以独立成一个搜索组件SearchGithub，代码清单：`app/containers/App/App.jsx`

```
import React from 'react';
import { SearchGithub } from '../../components'

const App = ({children, history}) => {
  return (
    <div className="main-container">
      <nav className="navbar navbar-default" role="navigation">
        <div className="col-sm-7 col-sm-offset-2" style={{marginTop: 15}}>
          <SearchGithub history={history}/>
        </div>
      </nav>
      <div className="container">
        {children}
      </div>
    </div>
  )
}

export default App

```

同时，我们要在components目录下新建一个SearchGithub组件
```
$ cd app/components && mkdir SearchGithub
$ cd SearchGithub && touch SearchGithub.jsx
```

封装我们的SearchGithub组件，代码清单：`app/components/SearchGithub/SearchGithub.jsx`
```
import React, { Component, PropTypes } from 'react';
import { browserHistory } from 'react-router';

class SearchGithub extends Component {
  static PropTypes = {
    history: PropTypes.object.isRequired
  }
  getRef(ref){
    this.usernameRef = ref;
  }
  handleSubmit(event){
    const username = this.usernameRef.value;
    this.usernameRef.value = '';

    const path = `/profile/${username}`;
    browserHistory.push(path)

  }
  render(){
    return (
      <div className="col-sm-12">
        <form onSubmit={() => this.handleSubmit()}>
          <div className="form-group col-sm-7">
            <input type="text" className="form-control" ref={(ref) => this.getRef(ref)} />
          </div>
          <div className="form-group col-sm-5">
            <button type="submit" className="btn btn-block btn-primary">搜索 Github</button>
          </div>
        </form>
      </div>
    )
  }
}

export default SearchGithub;


```

为了便于其他地方获取，我们需要更新components目录下的组件列表文件，代码清单：`app/components/index.js`
```
'use strict';

export TestDemo from './TestDemo/TestDemo.jsx';
export SearchGithub from './SearchGithub/SearchGithub.jsx';

```

哒哒，我们来看一下效果

<img src="/img/notetaker/router2.png" />

## 4.创建展示用户信息的Profile组件

前面的头部搜索框看起来像点样子，我们接着按照原型图来进行组件拆分。拆分之前，简单完善一下Home组件，这个组件是我们进入页面看到的默认的主页部分，简单写几个字示意一下即可。代码清单：`app/containers/Home/Home.jsx`

```
import React from 'react';

export default function Home () {
  return (
    <h2 className="text-center">
      通过 Github 用户名搜索代码资料
    </h2>
  )
}
```

理一下思路，按照原型图，我们需要创建一个容器组件，该组件可由展示用户基本信息、用户仓库信息、用户评论信息的三个组件组成，暂且将这个容器组件命名为Profile，Profile内的三个展示组件分别命名为UserProfile、UserRepos、Notes。

接下来进行创建

```
$ cd app/containers && mkdir Profile
$ cd Profile && touch Profile.jsx
```

完善Profile组件基本代码
```
import React, { Component } from 'react'

class Profile extends Component {
  render(){
    return (
      <div className="row">
        <div className="col-md-4">
          UserProfile 路由的参数是：{this.props.params.username}
        </div>
        <div className="col-md-4">
          UserRepos
        </div>
        <div className="col-md-4">
          Notes
        </div>
      </div>
    )
  }
}

export default Profile

```
更新组件列表文件，代码清单：`app/containers/index.js`

```
'use strict';

export App from './App/App.jsx';
export Home from './Home/Home.jsx';
export Profile from './Profile/Profile.jsx';

```

添加一条路由，代码清单：`app/routes/index.jsx`
```
import React from 'react';
import { Route, IndexRoute } from 'react-router';
import { App, Home, Profile } from '../containers';

export default (
  <Route path="/" component={App}>
    {/* 新加了profile路由 :后面是参数params */}
    <Route path="profile/:username" component={Profile} />
    <IndexRoute component={Home} />
  </Route>
)

```

这个时候再去输入路由参数的时候，我们可以在Profile组件中通过this.props.params.username拿到了。

## 5.创建组件UserProfile、UserRepos、Notes

创建三个组件

```
$ cd app/components && mkdir UserProfile UserRepos Notes
$ cd UserProfile && touch UserProfile.jsx
$ cd ../UserRepos && touch UserRepos.jsx
$ cd ../Notes && touch Notes.jsx
```

现在我们先简单的给这三个组件添加示意性的逻辑

代码清单：`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component } from 'react';

export default class UserRepos extends Component {
  render(){
    return (
      <div> UserRepos </div>
    )
  }
}

```

代码清单：`app/components/UserRepos/UserRepos.jsx`
```
import React, { Component } from 'react';

export default class UserRepos extends Component {
  render(){
    return (
      <div> UserRepos </div>
    )
  }
}

```

代码清单：`app/components/Notes/Notes.jsx`
```
import React, { Component } from 'react';

export default class Notes extends Component {
  render(){
    return (
      <div> Notes </div>
    )
  }
}

```

接下来更新一下组件维护列表，代码清单：`app/components/index.js`

```
'use strict';

export TestDemo from './TestDemo/TestDemo.jsx';
export SearchGithub from './SearchGithub/SearchGithub.jsx';
export UserProfile from './UserProfile/UserProfile.jsx';
export UserRepos from './UserRepos/UserRepos.jsx';
export Notes from './Notes/Notes.jsx';

```

ok，我们在Profile组件使用这三个组件，代码清单：`app/containers/Profile/Profile.jsx`

```
import React, { Component } from 'react';
import { UserProfile, UserRepos, Notes } from '../../components';

class Profile extends Component {
  render(){
    return (
      <div className="row">
        <div className="col-md-4">
          基本信息
          <UserProfile />
        </div>
        <div className="col-md-4">
          代码仓库
          <UserRepos />
        </div>
        <div className="col-md-4">
          笔记
          <Notes />
        </div>
      </div>
    )
  }
}

export default Profile

```

在浏览器中查看效果。

## 6.使用state和props传递数据

我们在Profile组件预设部分初始state，并且使用state和props将数据传递给三个子组件。

代码清单：`app/containers/Profile/Profile.jsx`

```
import React, { Component } from 'react';
import { UserProfile, UserRepos, Notes } from '../../components';

class Profile extends Component {
  state = {
    notes: ['1', '2', '3'],
    bio: {
      name: 'guoyongfeng'
    },
    repos: ['a', 'b', 'c']
  }
  render(){
    console.log(this.props);
    return (
      <div className="row">
        <div className="col-md-4">
          <UserProfile
            username={this.props.params.username}
            bio={this.state.bio} />
        </div>
        <div className="col-md-4">
          <UserRepos repos={this.state.repos} />
        </div>
        <div className="col-md-4">
          <Notes notes={this.state.notes} />
        </div>
      </div>
    )
  }
}

export default Profile

```

在三个子组件里面分别拿到这些数据进行展示。

代码清单：`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component } from 'react';

export default class UserProfile extends Component {
  render(){
    return (
      <div>
        <p> 基本信息 </p>
        <p> 姓名: {this.props.username} </p>
        <p> 介绍：{this.props.bio.name}</p>
      </div>
    )
  }
}

```

代码清单：`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component } from 'react';

export default class UserRepos extends Component {
  render(){
    return (
      <div>
        <p> GIT仓库 </p>
        <p> REPOS: {this.props.repos}</p>
      </div>
    )
  }
}

```

代码清单：`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component } from 'react';

export default class Notes extends Component {
  render(){
    return (
      <div>
        <p> 评论 </p>
        <p> Notes: {this.props.notes} </p>
      </div>
    )
  }
}

```

## 7.接入真实的数据

为了接入真实的数据，这里我们将用到几个新的知识点：
- [firebase](https://www.firebase.com/docs/web/quickstart.html)：是一个数据同步的云服务，帮助开发者开发具有「实时」（Real-Time）特性的应用，让我们实现真正的无后端编程，有木有很厉害，啊哈哈...
- reactfire: 一个react的mixin库，封装了六个组件公共的方法(bindAsArray,unbind,bindAsObject...)，专门用于处理React和Firebase集成的mixin方法，几行代码轻松获取数据，叼炸天...


首先下载这两个库
```
$ npm install --save reactfire firebase
```

然后在代码中引入：
```
import ReactFireMixin from 'reactfire';
import Firebase from 'firebase';
```

其中提到reactfire是一个mixin库，但是我们目前使用的是ES6语法写的，而不幸的是，ES6不支持mixin的写法，所以，我们只能想其他的办法，想知道更多请看[这里](http://zhuanlan.zhihu.com/purerender/20361937).

看完这篇文章后，我觉得我们可以用decorator来实现mixin的写法，比如在代码中这样写：
```
function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass {}

console.log(MyTestableClass.isTestable) // true
```

我们接着使用core-decorators提供的mixin来做重用模块的叠加，首先下载：
```
$ npm install --save core-decorators
```

然后在代码中引入这样就可以使用了
```
import { mixin } from 'core-decorators';

@mixin(ReactFireMixin)
class Profile extends Component {
 ...
}
```

这还没完成，我们目前使用的是decorator，但是浏览器不支持这个写法啊，怎么办，我们让babel来编译解决这个问题吧。首先需要下载一个能解析decorator的babel插件，然后在.babelrc里面配置：
```
$ npm install babel-plugin-transform-decorators-legacy --save-dev
```

配置.babrelrc
```
{
  "presets": ["es2015", "stage-0", "react"],
  "plugins": ["transform-runtime", "transform-decorators-legacy"]
}

```

so，目前为止，我们可以在代码中使用reactfire这个mixin类库以及decorator语法。

现在贴出Profile组件完整的代码：
```
import React, { Component } from 'react';
import { UserProfile, UserRepos, Notes } from '../../components';
import { mixin } from 'core-decorators';
import ReactFireMixin from 'reactfire';
import Firebase from 'firebase';

@mixin(ReactFireMixin)
class Profile extends Component {
  state = {
    notes: ['1', '2', '3'],
    bio: {
      name: 'guoyongfeng'
    },
    repos: ['a', 'b', 'c']
  }
  componentDidMount(){
    // 为了读写数据，我们首先创建一个firebase数据库的引用
    this.ref = new Firebase('https://github-note-taker.firebaseio.com/');
    // 调用child来往引用地址后面追加请求，获取数据
    var childRef = this.ref.child(this.props.params.username);
    // 将获取的数据转换成数组并且赋给this.state.notes
    this.bindAsArray(childRef, 'notes');
  }
  componentWillUnMount(){
    this.unbind('notes');
  }
  render(){
    return (
      <div className="row">
        <div className="col-md-4">
          <UserProfile
            username={this.props.params.username}
            bio={this.state.bio} />
        </div>
        <div className="col-md-4">
          <UserRepos
            username={this.props.params.username}
            repos={this.state.repos} />
        </div>
        <div className="col-md-4">
          <Notes
            username={this.props.params.username}
            notes={this.state.notes} />
        </div>
      </div>
    )
  }
}

export default Profile

```
