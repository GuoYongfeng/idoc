# github-notetaker应用开发

> 本次课程的完整代码在这里：https://github.com/GuoYongfeng/github-notetaker-app

## 0.介绍 INTRODUCTION

我们计划用最近所学的技术知识来完成一个完整且实用的功能应用，这个应用具有以下功能：

- 通过搜索github上的用户名来检索该用户的基本信息
- 可以检索到该用户的github上的所有代码仓库信息
- 可以对该用户进行简单的评论
- 通过特定路由可以访问特定用户的信息

大概是长这个样子

<img src="/img/notetaker/demo.png" />

在看到了这个原型图之后，我们来做一件很重要的事情，使用组件化的思维来解析这个应用需求。

<h3 style="text-align: center;">github-notetaker-app整体组件划分示意图</h3>

<img src="/img/notetaker/notetaker.png" />

## 1.启动
首先我们把脚手架项目下载下来并且安装启动
```
$ git clone git@github.com:GuoYongfeng/webpack-dev-boilerplate.git github-notetaker-app
$ cd github-notetaker-app && npm install
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

简单说一下react-router和history分别是什么：

- react-router is a complete routing library for React. react-router是一个专为react提供的完整路由库。
- history is a JavaScript library that lets you easily manage session history anywhere JavaScript runs. history是一个js库，可以让您轻松地管理会话历史。

接下来我们开始写代码：

代码清单：`app/index.js`
```
import React from 'react';
import ReactDOM from 'react-dom';
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
import { App, Home, About } from '../containers';

export default (
  <Route path="/" component={App}>
    <IndexRoute component={Home} />
    <Route path="about" component={About} />
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
$ cd app/containers && mkdir Home About
$ cd Home && touch Home.jsx
$ cd ../About && touch About.jsx
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

代码清单：`app/containers/Home/Home.jsx`
```
import React, { Component } from 'react';

class About extends Component {
  render() {
    return (
      <h2>content from About Component</h2>
    );
  }
}

export default About;

```

这里我们新增了组件，同时在containers下面的组件索引文件中进行更新。
代码清单：`app/containers/index.js`
```
'use strict';

export App from './App/App.jsx';
export Home from './Home/Home.jsx';
export About from './About/About.jsx';
```

初步完成路由的管理，我们先在命令行窗口停止服务，需要修改`package.json`文件，在启动webpack-dev-server的时候加上参数`--history-api-fallback`

> 因为我们这里用的是browserHistory，启动服务的时候加上history-api-fallback用于`enables support for history API fallback.`

好了，重新运行`npm run dev`，我们接下来在浏览器中查看效果.

<img src="/img/notetaker/router1.png" />


## 3.新增头部搜索组件

在新增组件之前，为了快速的写出好看的页面，我们考虑把bootstrap引入使用。

下载bootstrap

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

现在贴出Profile组件完整的代码：`app/containers/Profile/Profile.jsx`
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

现在通过firebase获取的是notes的数据，我们到Notes组件来看一下传过来的是什么数据.
代码清单：`app/components/Notes/Notes.jsx`

```
import React, { Component } from 'react';

export default class Notes extends Component {
  render(){
    console.log('notes:', this.props.notes);

    return (
      <div>
        <p> 评论 </p>
      </div>
    )
  }
}

```

数据是长这样子的：

<img src="/img/notetaker/firebase.png" />

## 8.评论列表组件NoteList

评论的数据拿到了，我们创建一个展示评论数据的组件NoteList，并且在Notes组件中调用
```
$ cd app/components/Notes && touch NoteList.jsx
```

ok，我们在Notes组件中使用NoteList，并将获取的notes数据传给它；
```
import React, { Component } from 'react';
import NoteList from './NoteList.jsx';

export default class Notes extends Component {
  render(){
    console.log('notes:', this.props.notes);

    return (
      <div>
        <p> 对{this.props.username}评论： </p>
        <NoteList notes={this.props.notes} />
      </div>
    )
  }
}

```

然后封装NoteList组件
```
import React, { Component } from 'react';

export default class NoteList extends Component {
  render(){
    let notes = this.props.notes.map((note, index) => {
      return <li className="list-group-item" key={index}>{note['.value']}</li>
    })
    return (
      <ul className="list-group">
        {notes}
      </ul>
    )
  }
}

```

现在，我们可以在浏览器中看到展示真实数据的Notes组件

<img src="/img/notetaker/notelist.png" />

## 9.为组件添加PropTypes接口约束

代码清单；`app/components/Notes/Notes.jsx`

```
import React, { Component, PropTypes } from 'react';
import NoteList from './NoteList.jsx';

export default class Notes extends Component {
  static propTypes = {
    username: PropTypes.string.isRequired,
    notes: PropTypes.array.isRequired
  }
  render(){
    console.log('notes:', this.props.notes);

    return (
      <div>
        <p> 对{this.props.username}评论： </p>
        <NoteList notes={this.props.notes} />
      </div>
    )
  }
}

```

代码清单；`app/components/UserRepos/UserRepos.jsx`

```
import React, { Component, PropTypes } from 'react';

export default class UserRepos extends Component {
  static propTypes = {
    username: PropTypes.string.isRequired,
    repos: PropTypes.array.isRequired
  }
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

代码清单；`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component, PropTypes } from 'react';

export default class UserProfile extends Component {
  static propTypes = {
    username: PropTypes.string.isRequired,
    bio: PropTypes.object.isRequired
  }
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

## 10.使用axios请求github API的数据

> [axios](https://github.com/mzabriskie/axios): Promise based HTTP client for the browser and node.js
> axios是一个能同时运行于浏览器端和nodejs的AJAX/HTTP方法/库


我们将用axios来请求github API的接口数据，用于组件的展示。下载：
```
$ npm install --save axios
```

首先写一个公共的工具函数来请求github上用户信息和仓库信息相关的数据。

```
$ cd app && mdkir util
$ cd util && touch helper.js
```

代码清单：`app/util/helper.js`
```
import axios from 'axios'

// axios用法很简单，请参考这里：https://github.com/mzabriskie/axios

/**
 * 传入用户名，获取用户的github上仓库信息
 * @param  {[type]} username [description]
 * @return {[type]}          [description]
 */
function getRepos(username){
  // 这里使用了 ES6 的字符串模板
  return axios.get(`https://api.github.com/users/${username}/repos`);
}

/**
 * 传入用户名，获取用户github上的基本信息
 * @param  {[type]} username [description]
 * @return {[type]}          [description]
 */
function getUserInfo(username){
  return axios.get(`https://api.github.com/users/${username}`);
}

export default function getGithubInfo(username){
  // 将请求回来的数据做了一个 merge 操作
  return axios.all([
    getRepos(username),
    getUserInfo(username)
  ])
  .then((arr) => ({
      repos: arr[0].data,
      bio: arr[1].data}
  ));
}
```

> 上面的代码中，我们使用axios发送请求到`https://api.github.com`，如果不太清楚通过github API获取数据的话，没关系，请到这里看一下：`https://developer.github.com/v3`。同时访问链接感受下：`https://api.github.com/users/guoyongfeng`。

现在我们在Profile组件中引入这个工具函数，并且传入用户名，查看返回的数据。

```
import React, { Component } from 'react';
import { UserProfile, UserRepos, Notes } from '../../components';
import { mixin } from 'core-decorators';
import ReactFireMixin from 'reactfire';
import Firebase from 'firebase';
import getGithubInfo from '../../util/helper';

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

    getGithubInfo( this.props.params.username )
      .then( ( data ) => {
        // 测试一下传入用户名后返回的数据
        console.log( data );
        // 更新state数据
        this.setState({
          bio: data.bio,
          repos: data.repos
        })
      });
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

这里运行的时候会有个报错，以为之前在UserRepos组件中有这样一行代码：
```
<p> REPOS: {this.props.repos}</p>
```
repos是一个对象数组，直接展示会报错，暂时先去掉这行代码。

然后看浏览器返回的数据。

<img src="/img/notetaker/github-data.png" />

## 11.将数据传入组件进行展示

从第10步我们已经妥妥的把数据取回来了，那么接下来的事情将变得轻松，我们这需要将数据传入组件，修改组件的UI进行展示即可。甚至，你完全可以做个和github官网一样漂亮的首页。let's do it....

首先，来完善一下UserRepos组件吧
代码清单：`app/components/UserRepos/UserRepos.jsx`
```
import React, { Component, PropTypes } from 'react';

export default class UserRepos extends Component {
  static propTypes = {
    username: PropTypes.string.isRequired,
    repos: PropTypes.array.isRequired
  }
  render(){
    console.log('repos:', this.props.repos);

    let repos = this.props.repos.map( (repo, index ) => {
      return (
        <li className="list-group-item" key={index}>
          {repo.html_url && <h4><a href={repo.html_url}>{repo.name}</a></h4>}
          {repo.description && <p>{repo.description}</p>}
        </li>
      )
    });

    return <div>
      <h3> 用户的 Git 仓库 </h3>
      <ul className="list-group">
        {repos}
      </ul>
    </div>
  }
}

```

然后，完善一下UserProfile组件，代码清单：`app/components/UserProfile/UserProfile.jsx`

```
import React, { Component, PropTypes } from 'react';

export default class UserProfile extends Component {
  static propTypes = {
    username: PropTypes.string.isRequired,
    bio: PropTypes.object.isRequired
  }
  render(){
    let bio = this.props.bio;
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

```

查看一下效果吧

<img src="/img/notetaker/last.png" />

## 结语

完整的示例代码在这里：https://github.com/GuoYongfeng/github-notetaker-app

本次课程内容比较丰富，请按课程步骤执行操作，遇到问题请请issue：https://github.com/GuoYongfeng/github-notetaker-app/issues
