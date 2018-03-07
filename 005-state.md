介绍一下 react 最核心的概念 state ，也就是状态值。以及 lifecycle ，生命周期函数。

## 困扰

目前为止我们只有一种更新 UI 的方式。

就是用 `ReactDOM.render()` 来修改渲染到页面上的内容

```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  )
  ReactDOM.render(element, document.getElementById('root'))
}

setInterval(tick, 1000)
```

下面我们要学会真正的把 `Clock` 组件封装好，让它可复用。让它自己设置好自己的 timer 定时器，并且每秒钟更新一下自己。

先来封装钟表的界面

```js
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  )
}

function tick() {
  ReactDOM.render(<Clock date={new Date()} />, document.getElementById('root'))
}

setInterval(tick, 1000)
```

至此，还有一项重点需求没有达成：`Clock` 应该可以设置自己的定时器，并且每秒钟自动更新自己。这些代码都应该写到 `Clock` 组件内部的。

## state

理想情况下，我们应该可以直接写下面的语句，然后 `Clock` 可以自己去更新自己

```js
ReactDOM.render(<Clock />, document.getElementById('root'))
```

实现这个就需要在 `Clock` 组件中引入 state 了。

state 状态值，跟 props 属性， 很类似，不过 state 是私有的，是完全被组件控制的。

之前提过一下，class 式组件要比函数式组件功能多，可以使用本地 state 就是只能用在 class 式组件中的功能。

## 把函数转换成 class

把 `Clock` 这样的函数式组件转换成 class 需要下面五步：

* 创建一个 ES6 的 class ，名字不变，继承自 React.Component 。
* 里面添加一个 `render()` 函数。
* 函数主体内容添加到 `render()` 内部。
* `render()` 里的 `props` 都改成 `this.props`
* 删除已经为空的的那个函数声明

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}
```

现在 `Clock` 就是 class 式组件了。这样就可以使用本地 state 和声明周期函数这些功能了。

## class 中添加本地 state

把 `date` 从 props 变成 state 需要三步：

1.  把 `this.props.date` 替换成 `this.state.date`

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}
```

2.  添加类的构造函数，constructor ，来给 `this.state` 赋初始值

```js
class Clock extends React.Component {
  constructor(props) {
    super(props)
    this.state = { date: new Date() }
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}
```

注意我们如何把 `props` 传递给父组件的构造函数的：

```js
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

Class 组件都是需要呼叫父组件的构造函数，并且传入 `props` 作为参数的。

3.  删除 `<Clock />` 的 `date` 属性。

```js
ReactDOM.render(<Clock />, document.getElementById('root'))
```

稍后我们会把定时器组件添加到组件中。

结果最后是这样

```js
class Clock extends React.Component {
  constructor(props) {
    super(props)
    this.state = { date: new Date() }
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}

ReactDOM.render(<Clock />, document.getElementById('root'))
```

接下来，让 `Clock` 设置自己的的定时器并且每秒钟自动更新。

## 给 class 添加生命周期方法

生命周期函数，生命周期方法，都是一回事，后面咱们就不做区分了。

在一个包含很多组件的大项目中，如果组件不再用了，一定要注意释放资源。

下面我想要在每次 `Clock` 首次渲染到 DOM 中的时候设置一个定时器。这个过程被叫做 React 组件的挂载（ mounting ）。

同时我想要在 `Clock` 从 DOM 中被移除的时刻清除定时器。这个过程叫做卸载（ unmounting ）。

我们可以定义一些特殊的方法，里面的代码会在组件挂载和卸载的时候自动被执行：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props)
    this.state = { date: new Date() }
  }

  componentDidMount() {}

  componentWillUnmount() {}

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}
```

这些方法被叫做”生命周期钩子函数“（ lifecyle hooks ） ，也就是咱们说的生命周期方法。

先看 `componentDidMount` 这个钩子，它会在组件被渲染到 DOM 中之后执行。所以这里设置定时器是最适合的：

```js
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

注意，我们把定时器 ID 直接保存到了 this 上。

尽管 `this.props` 是 React 自带的，`this.state` 也是有特殊含义的，咱们仍然可以自由的添加新的字段到 class 里面来。

`render()` 中不会用到的东西，不应该添加到 state 中。

`componentWillUnmount()` 中来清除定时器

```js
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```

最后，我们来定义一个 `tick()` 函数，让 `Clock` 组件每秒钟都去执行一次。

`tick()` 中用 `this.setState` 来更新组件的本地 state ：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props)
    this.state = { date: new Date() }
  }

  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000)
  }

  componentWillUnmount() {
    clearInterval(this.timerID)
  }

  tick() {
    this.setState({
      date: new Date()
    })
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    )
  }
}

ReactDOM.render(<Clock />, document.getElementById('root'))
```

现在 clock 没秒钟都会 `tick` 一次。

来整理一下思路，也关注一下各个方法的执行顺序：

1.  当把 `<Clock />` 传递给 `ReactDOM.render()` 的时候，React 执行了 `Clock` 组件的构造函数。因为 `Clock` 需要展示当前时间，所以它把一个保护当前时间的对象赋值给了 `this.state` 。后续，我们会更新这个 state 值。

2.  React 这时就会执行 `Clock` 组件的 `render` 方法。这样 React 就知道屏幕上应该显示什么内容了。React 会根据 `render` 方法的输出更新 DOM 。

3.  当 `Clock` 的输出内容被插入 DOM 中后，React 会执行 `componentDidMount` 生命周期钩子。里面 `Clock` 组件会让浏览器设置一个定时器，以便每秒钟执行一次 `tick` 。

4.  每秒钟 `tick` 都会被浏览器调用一次。其中，`Clock` 组件会调用 `setState()`来更新 UI 。`setState()` 一执行，React 就会知道 state 值已经发生了变化，于是就会再次执行 `render` 把新内容显示到屏幕上。这次，`render` 里面的 `this.state.date` 就跟最初不同了，所以最终显示出来的也是更新后的状态。

5.  一旦 `Clock` 组件被从 DOM 中移除，`componentWillUnmount` 就会被执行，定时器也就会被停止了。

## 正确使用 state

使用 `setState` 有三大注意点。

1.  不要直接修改 state

例如下面的做法并不会让组件重新渲染：

```js
// Wrong
this.state.comment = 'Hello';
Instead, use setState():
```

下面的做法才是正确的：

```js
// Correct
this.setState({ comment: 'Hello' })
```

直接给 `this.state` 赋值的地方只有一处，就是在构造函数中。

2.  状态更新可能是异步的

React 可能会出于性能考虑，把好几个 `setState` 捆绑到一起执行。

因为 `this.props` 和 `this.state` 都可能是异步更新的，所以不应该基于它们的值去运算下一个状态值。

例如，下面的代码可能不能正确更新计数器：

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment
})
```

要解决这个问题，可以使用 `setState` 的另外一种形式，用函数而不是对象作为参数。函数中接收之前的 state 作为第一个参数，把属性值作为第二个参数：

```js
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}))
```

3.  状态值更新会被融合

当咱们执行 `setState` 的时候，React 会把参数中的对象融合到当前 state 中。

例如，state 中可能会包含多个独立的变量：

```js
constructor(props) {
  super(props)
  this.state = {
    posts: [],
    comments: []
  }
}
```

我们可以独立的去更新每一个

```js
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

可以想象到的是，更新一个，不会影响另一个的。

## 参考

* https://reactjs.org/docs/state-and-lifecycle.html
