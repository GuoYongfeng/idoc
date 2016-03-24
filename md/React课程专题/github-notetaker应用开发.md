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

## 5.UserProfile UserRepos Notes

创建三个组件

```
$ cd app/components && mkdir UserProfile UserRepos Notes
$ cd UserProfile && touch UserProfile.jsx
$ cd ../UserRepos && touch UserRepos.jsx
$ cd ../Notes && touch Notes.jsx
```
