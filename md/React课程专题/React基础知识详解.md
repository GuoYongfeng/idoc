
<h1 style="font-size: 40px;text-align:center;color: blue;">React基础知识详解</h1>

> **编者按：**除组件化、虚拟DOM在复用以及性能上带来的一般好处外，React思想使得开发者之间更好的分工与合作，在配合上非常顺畅，规范的接口以及极强的约束，使得整个代码结构清晰，不同开发者的代码高度一致。


## 0. 本次课程大纲


> **课程主线如下：**

- React是什么
react 是一个做 UI 的库，具体来说是做 UI 组件的库，专注于做 mvc 中的 v.
- 为什么学React
api 少，类库易学
组件内聚，易于组合
原生组件和自定义组件融合渲染
状态/属性驱动全局更新，不用关注细节更新
ES6 webpack ... 生态圈/工具链完善
- 怎么用React（`第1-9部分内容`）
- 搭建示例应用（`第10部分内容`）


## 1. React相关文件顶层api的介绍

最简单的React组件及其渲染
```
import React, {Component} from 'react';
import ReactDOM from 'react-dom';

class Hello extends Component {
    render() {
        return <div>Hello {this.props.name}</div>;
    }
}

ReactDOM.render(<Hello name="World" />, document.getElementById('container'));

```

### react.js
React.Component 使用ES6的class创建组件用的API
React.createClass 使用ES5的class创建组件用的API
React.PropTypes
React.Children 操作 map/forEach children 工具类

还有几个不是很常用的
React.createElement
React.cloneElement
React.createFactory
React.DOM

### react-dom.js
ReactDOM.render 渲染组件到 dom
ReactDOM.unmountComponentAtNode
ReactDOM.findDOMNode

### react-dom-server.js
ReactDOMServer.renderToString
ReactDOMServer.renderToStaticMarkup

## 2. jsx语法
---
类似 xml 的语法，用来描述组件树
```
<div classname="x">
  <a href="#">#</a>
    <component x="y">1</component>
</div>
```
不用JSX，用React提供的API写的话
```
React.createElement('div',{
  className:'x'
    },[
    React.createElement('a',{href:'#'},'#'),
    React.createElement(Component,{x:'y'},1);
]);
```
注意点

### 2.1 注释
```
var content = (
  <Nav>
    {/* 一般注释, 用 {} 包围 */}
    <Person
      /* 多
         行
         注释 */
      name={window.isLoggedIn ? window.name : ''} // 行尾注释
    />
  </Nav>
);
```
### 2.2 styles
```
var styles = {
	borderColor: '#ccc',
	fontSize: '18px'
};

ReactDOM.render(<div style={styles}></div>, container);
```
### 2.3 顶层一个根元素
```
// 这样会报错
render(){
	return (
		<h2>nn</h2>
	);
}
// 应该顶层只有一个元素
render(){
	return (
		<h1>hah</h1>
	);
}
```
### 2.4 组件名按驼峰命名
```
import Button from './Button.js';

...
render(){
	return <Button />
}
...
```
<Button />
### 2.5 属性名不能和 js 关键字冲突

例如：className, readOnly, htmlfor
### 2.6  JSX 嵌入变量
可以通过 {变量名} 来将变量的值作为属性值
```
render(){
	return <div className={this.state.isComplete ? 'is-complete' : ''}>
		hello world
	</div>
}
```
### 2.7  JSX SPREAD
可以用通过 {...obj} 来批量设置一个对象的键值对到组件的属性，注意顺序
```
  var props = {};
  props.foo = x;
  props.bar = y;
  var component = <Component {...props} />;
```
## 3. 数据流：state props propType

### 3.1 state
用状态控制组件变化
可以把一个组件看做一个状态机, 每一次状态对应于组件的一个 ui

**组件内部的状态，可以使用 state**

```
var Timer = React.createClass({
  getInitialState: function() {
    return {secondsElapsed: 0};
  },
  tick: function() {
    this.setState({secondsElapsed: this.state.secondsElapsed + 1});
  },
  componentDidMount: function() {
    this.interval = setInterval(this.tick, 1000);
  },
  componentWillUnmount: function() {
    clearInterval(this.interval);
  },
  render: function() {
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
});

React.render(<Timer />, document.getElementById('container'));
```
setstate 哪些应该作为state

### 3.2 props
通过 this.props 可以获取传递给该组件的属性值，还可以通过定义 getDefaultProps 来指定默认属性值

注意：默认值getDefaultProps为所有组件实例共享的

props.children
props.map/filter
传递进去的
区别，传递数据的例子，封装APi
组件树数据传递

```
var B = React.createClass({
    getDefaultProps(){
        return {
            title:'default'
        };
    },
    render(){
        return <b>{this.props.title}</b>
    }
});

React.render(<div>
<B title="指定  "/> <B/>
</div>,document.getElementById('container'));
```

### 3.3 propTypes
通过指定 propTypes 可以校验属性值的类型
注意：校验仅为提升开发者体验
```
var B = React.createClass({
    propTypes: {
        title: React.PropTypes.string,
    },

    getDefaultProps(){
        return {
            title:'default'
        };
    },
    render(){
        return <b>{this.props.title}</b>
    }
});

React.render(<div>
<B title="指定  "/> <B title={2}/>
</div>,document.getElementById('container'));
```

## 4. 调用API定义组件

用 React.createClass 定义组件时允许传入相应的配置，包括组件生命周期提供的一系列钩子函数。

### 4.1 组件初始定义

getDefaultProps 得到默认属性对象，这个在ES6的时候不需要这样定义
propTypes 属性检验规则
mixins 组件间公用方法


### 4.2 初次创建组件时调用

getInitialState 得到初始状态对象
render 返回组件树. 必须设置
componentDidMount 渲染到 dom 树中是调用，只在客户端调用，可用于获取原生节点

### 4.3 组件的属性值改变时调用

componentWillReceiveProps 属性改变调用
shouldComponentUpdate 判断是否需要重新渲染
render 返回组件树. 必须设置
componentDidUpdate 渲染到 dom 树中是调用, 可用于获取原生节点

### 4.4 销毁组件
最后是 componentWillUnmount 组件从 dom 销毁前调用

### 4.5 示例诠释组件全生命周期

```
function log(str){
  document.getElementById('log').innerHTML+='<p>'+str+'</p>';;
}
document.getElementById('clear').onclick=function(){
document.getElementById('log').innerHTML='';
};
var Test = React.createClass({
  getInitialState() {
    log('getInitialState');
    return {
      value: this.props.value
    };
  },

  componentWillReceiveProps(nextProps){
    log('componentWillReceiveProps');
    this.setState({
    	value: nextProps.value
    });
  },

  shouldComponentUpdate(nextProps,nextState){
    log('shouldComponentUpdate');
    return true;
  },

  componentWillUpdate(nextProps,nextState){
    log('componentWillUpdate');
  },

  componentWillMount(){
    log('componentWillMount');
  },

  render() {
    log('render');
    return <span>{this.props.value}</span>
  },

  componentDidMount() {
  	log('componentDidMount');
  },

  componentDidUpdate(prevProps,prevState) {
  	log('componentDidUpdate');
  },

  componentWillUnmount(prevProps,prevState) {
  	log('componentWillUnmount');
  }
});


var Hello = React.createClass({
    getInitialState() {
      return {
        value:1,
        destroyed:false
      };
    },
    increase() {
    	this.setState({
        	value: this.state.value+1
        });
    },
    destroy() {
    	this.setState({
        	destroyed: true
        });
    },
    render: function() {
    	if(this.state.destroyed){
        	return null;
        }
        return <div>
        <p>
          <button onClick={this.increase}>increase</button>
          <button onClick={this.destroy}>destroy</button>
        </p>
        <Test value={this.state.value}/>
        </div>;
    }
});

React.render(<Hello />, document.getElementById('container'));

```

### 4.6 回顾组件的渲染过程
```
# 创建-》渲染-》销毁

getDefaultProps()
getInitialState()
componentWillMount()
render()
componentDidMount()
componentWillUnmount()

# 更新组件

componentWillReceiveProps()
shouldComponentUpdate()
componentWillUpdate()
render()
componentDidUpdate()
```
## 5. 使用ref对操作DOM

该功能是为了结合现有非 react 类库，通过 ref/refs 可以取得组件实例，进而取得原生节点
注意：尽量通过 state/props 更新组件，不要使用该功能

```
var Test = React.createClass({
    componentDidMount(){
      alert(React.findDOMNode(this.refs.content).innerHTML);
    },
    render(){
        return <div>
            <h3>header</h3>
            <div ref='content'>content</div>
        </div>;
    }
});

React.render(<Test />,document.getElementById('container'));
```
## 6. 事件event

可以通过设置原生 dom 组件的 onEventType 属性来监听 dom 事件，例如 onClick, onMouseDown，在加强组件内聚性的同时，避免了传统 html 的全局变量污染
```
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

React.render(
  <LikeButton />,
  document.getElementById('container')
);
```

注意：事件回调函数参数为标准化的事件对象，可以不用考虑 ie

## 7. 组件的组合

### 7.1 使用自定义的组件
```
var A = React.createClass({
    render(){
        return <a href='#'>a</a>
    }
});

var B = React.createClass({
    render(){
        return <i><A /> !</i>;
    }
});

React.render(<B />,document.getElementById('container'));
```
### 7.2 组合 CHILDREN
自定义组件中可以通过 this.props.children 访问自定义组件的子节点
```
var B = React.createClass({
    render(){
        return <ul>
            {React.Children.map(this.props.children,function(c){
                return <li>{c}</li>;
            })}
        </ul>;
    }
});

React.render(<B>
<a href="#">1</a>
2
</B>,document.getElementById('container'));
```
## 8. form表单操作

和 html 的不同点：

value/checked 属性设置后，用户输入无效
textarea 的值要设置在 value 属性
select 的 value 属性可以是数组，不建议使用 option 的 selected 属性
input/textarea 的 onChange 用户每次输入都会触发（即使不失去焦点）
radio/checkbox 点击后触发 onChange

- 受控组件
如果设置了 value 属性，那么改组件变为受控组件，用户无法输入，除非程序改变 value 属性

```
var Test = React.createClass({
    render(){
        return <input value="x" />
    }
});

React.render(<Test />,document.getElementById('container'));
```

可以通过监听 onChange 事件结合 state 来改变 input 的值

```

```
- 不受控组件
设置 defaultValue 为设置 input 的初始值，之后 input 的值由用户输入

```
var Test = React.createClass({
    render(){
        return <input defaultValue="xyz" />
    }
});

React.render(<Test />,document.getElementById('container'));
```

## 9. mixin共享

mixin 是一个普通对象，通过 mixin 可以在不同组件间共享代码
```
var mixin = {
    propTypes: {
        title: React.PropTypes.string,
    },

    getDefaultProps(){
        return {
            title:'default'
        };
    },
};

var A = React.createClass({
    mixins: [mixin],
    render(){
        return <i>{this.props.title}</i>
    }
});

var B = React.createClass({
    mixins: [mixin],
    render(){
        return <b>{this.props.title}</b>
    }
});

React.render(<div>
<B/> <A title={2}/>
<A/>
</div>,document.getElementById('container'));
```

## 10. 一步步搭建一个评论的应用

### 10.1 功能描述
评论的列表
提交表单
后端结合

### 10.2 组件分解
顶层 CommentBox
评论列表 CommentList
单条评论 Comment
评论表单 CommentForm

### 10.3 最简单的外层组件
```
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('container')
);
```
### 10.4 组合组件
```
var CommentList = React.createClass({
  render: function() {
    return (
      <div className="commentList">
        Hello, world! I am a CommentList.
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function() {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList />
        <CommentForm />
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('container')
);
```
### 10.5 属性传递
```
var Comment = React.createClass({
  render: function() {
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        {this.props.children}
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function() {
    return (
      <div className="commentList">
        <Comment author="作者 1">评论 1</Comment>
        <Comment author="作者 2">评论 2</Comment>
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function() {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList />
        <CommentForm />
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('container')
);
```
### 10.6 使用 DOM 库 MARKED
```
var Comment = React.createClass({
  render: function() {
  var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function() {
    return (
      <div className="commentList">
        <Comment author="作者 1">评论 1</Comment>
        <Comment author="作者 2"> *评论 2* </Comment>
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function() {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList />
        <CommentForm />
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('container')
);
```
### 10.7 数据分离
```
var data = [
  {author: "作者 1", text: "评论 1"},
  {author: "作者 2", text: "*评论 2*"}
];

var Comment = React.createClass({
  render: function() {
  var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function() {
    var commentNodes = this.props.data.map(function (comment) {
      return (
        <Comment author={comment.author}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function() {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={data}/>
        <CommentForm />
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('container')
);
```
### 10.8 从服务器取得数据
```
var Comment = React.createClass({
  render: function () {
    var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function () {
    var commentNodes = this.props.data.map(function (comment) {
      return (
        <Comment author={comment.author}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});

var CommentForm = React.createClass({
  render: function () {
    return (
      <div className="commentForm">
        Hello, world! I am a CommentForm.
      </div>
    );
  }
});

var CommentBox = React.createClass({
  loadCommentsFromServer: function () {
    var self = this;
    new Request.JSON({
      url: this.props.url,
      data: {
        json: JSON.encode({
          data: [
            {
              author: '作者 1',
              text: '评论 1,' +Date.now()
            },
            {
              author: '作者 2',
              text: ' *评论 2,'+Date.now()+'* '
            }
          ]
        }),
        delay:1
      },
      onSuccess(res) {
        self.setState({data: res.data})
      }
    }).send();
  },
  getInitialState: function () {
    return {data: []};
  },
  componentDidMount: function () {
    this.loadCommentsFromServer();
    setInterval(this.loadCommentsFromServer, this.props.pollInterval);
  },
  render: function () {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm />
      </div>
    );
  }
});

React.render(
  <CommentBox url="/echo/json/" pollInterval={2000} />,
  document.getElementById('container')
);
```
### 10.9 评论表单
```
var Comment = React.createClass({
  render: function () {
    var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function () {
    var commentNodes = this.props.data.map(function (comment) {
      return (
        <Comment author={comment.author}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});

var CommentForm = React.createClass({
  getInitialState() {
    return {
      name: '',
      text: ''
    }
  },
  updateField(field, e) {
  console.log(e);
    var state = {};
    state[field] = e.target.value;
    this.setState(state);
  },
  handleSubmit(e){
    e.preventDefault();
    this.props.onPost({
      name:this.state.name,
      text:this.state.text
    });
    this.setState({
      name:'',
      text:''
    });
  },
  render: function () {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input placeholder="Your name" value={this.state.name} onChange={this.updateField.bind(this, 'name')}/>
        <input placeholder="Say something..."
          value={this.state.text} onChange={this.updateField.bind(this, 'text')}
        />
        <input type="submit" value="Post" />
      </form>
    );
  }
});

var database=[
  {
    author: '作者 1',
    text: '评论 1,' + Date.now()
  },
  {
    author: '作者 2',
    text: ' *评论 2,' + Date.now() + '* '
  }
];

var CommentBox = React.createClass({
  loadCommentsFromServer: function () {
    var self = this;
    $.ajax({
      url: this.props.url,
      method:'post',
      dataType:'json',
      data: {
        json:JSON.stringify({
          data:database
        })
      },
      success(res) {
      console.log(res)
        self.setState({data: res.data})
      }
    });
  },
  getInitialState: function () {
    return {data: []};
  },
  handlePost(){

  },
  componentDidMount: function () {
    this.loadCommentsFromServer();
  },
  render: function () {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm onPost={this.handlePost}/>
      </div>
    );
  }
});



React.render(
  <CommentBox url="/echo/json/" />,
  document.getElementById('container')
);
```
### 10.10 最后：通知重新渲染
```
var Comment = React.createClass({
  render: function () {
    var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return (
      <div className="comment">
        <h2 className="commentAuthor">
          {this.props.author}
        </h2>
        <span dangerouslySetInnerHTML={{__html: rawMarkup}} />
      </div>
    );
  }
});

var CommentList = React.createClass({
  render: function () {
    var commentNodes = this.props.data.map(function (comment) {
      return (
        <Comment author={comment.author}>
          {comment.text}
        </Comment>
      );
    });
    return (
      <div className="commentList">
        {commentNodes}
      </div>
    );
  }
});

var CommentForm = React.createClass({
  getInitialState() {
    return {
      name: '',
      text: ''
    }
  },
  updateField(field, e) {
    var state = {};
    state[field] = e.target.value;
    this.setState(state);
  },
  handleSubmit(e){
    e.preventDefault();
    this.props.onPost({
      author:this.state.name,
      text:this.state.text
    });
    this.setState({
      name:'',
      text:''
    });
  },
  render: function () {
    return (
      <form className="commentForm" onSubmit={this.handleSubmit}>
        <input placeholder="Your name" value={this.state.name} onChange={this.updateField.bind(this, 'name')}/>
        <input placeholder="Say something..."
          value={this.state.text} onChange={this.updateField.bind(this, 'text')}
        />
        <input type="submit" value="Post" />
      </form>
    );
  }
});

var database=[
  {
    author: '作者 1',
    text: '评论 1,' + Date.now()
  },
  {
    author: '作者 2',
    text: ' *评论 2,' + Date.now() + '* '
  }
];

var CommentBox = React.createClass({
  loadCommentsFromServer: function () {
    var self = this;
    $.ajax({
      url: this.props.url,
      method:'post',
      dataType:'json',
      data: {
        json:JSON.stringify({
          data:database
        })
      },
      success(res) {
        self.setState({data: res.data})
      }
    });
  },
  getInitialState: function () {
    return {data: []};
  },
  handlePost(post){
    database.push(post);
    this.loadCommentsFromServer();
  },
  componentDidMount: function () {
    this.loadCommentsFromServer();
  },
  render: function () {
    return (
      <div className="commentBox">
        <h1>Comments</h1>
        <CommentList data={this.state.data} />
        <CommentForm onPost={this.handlePost}/>
      </div>
    );
  }
});



React.render(
  <CommentBox url="/echo/json/" />,
  document.getElementById('container')
);
```

## 基础部分完结寄语

总结起来，学习的难度不高，以上内容掌握后，基本能够进行React的开发，后续我们继续相关内容的讲解。
