# React技术学习--基础篇（一）

> 越是基础的东西，越是重要；越是原理的内容，越要去理清楚。为什么在开篇回去强调这个内容，因为也许等你学习React后会发现，使用React的人慢慢的分成了两派，一派是专门封装React组件或组件库的，一派是专门使用前者的。所以，学习基础将是让自己拥有封装和运用组件的基础。

## JSX语法

React的精髓在于组件，组件的基础是jsx，所以好好学吧。先不讲道理，直接上代码：

```javascript
<script type="text/jsx">
  var div = React.createClass({
    render: function(){
      return (
        <div>
          <h3>hello wold</h3>
        </div>
      );
    }
  });

</script>
```


## 组件生命周期中的那些钩子函数

学会了编写组件，则接下来不得不学习组件的生命周期，以及每个生命周期对应的一些钩子函数：
- 虚拟期
  * getInitialState()
  * getDefaultProps()
  * componentWillmount()
- 运行期
  * componentDidMount()
  * shouldComponentUpdate()
  * componentWillUpdate()
  * componentWillUnmount()
- 销毁期
  * componentDidUnmount()
## state和prop为基础的单向数据流

需要理解清楚的是，虽然state和prop都是存储数据的，但是要区分二者的区别：

- state存放的是流动的，变化的组件数据
- prop则一般用来存放组件初始后不变的数据和属性

二者的结合则可完成组件的单向数据流动

## 复合组件

多个简单的组件嵌套，可构成一个复杂的复合总结，从而完成复杂的交互逻辑

## DOM操作和事件处理


