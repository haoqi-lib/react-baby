React 处理事件跟 DOM 元素处理事件很类似。

## 语法上有差异

跟传统的 DOM 事件对比，还是有些语法差异的：

* React 事件都是骆驼字体，不是全部小写
* JSX 里面传递事件是传函数，而不是传字符串

例如，HTML DOM 下这么写：

```html
<button onclick="activateLasers()">Activate Lasers</button>
```

React 的 JSX 里面就这么写了：

```js
<button onClick={activateLasers}>Activate Lasers</button>
```

另外一个差异就是 return false 去阻止默认行为这种方式不能用在 React 这里。必须使用 `preventDefault` 。

例如，HTML 里面：

```html
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```

这里 `return false` 可以用来阻止默认行为，也就是打开连接指向的页面。

React 中就要写成：

```js
function ActionLink() {
  function handleClick(e) {
    e.preventDefault()
    console.log('The link was clicked.')
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  )
}
```

这里， `e` 就是一个事件（ event ）。React 定义这些事件根据的是 W3C 的规定，所以无需担心浏览器兼容问题。

用上了 React ，`addEventListerner` 就基本不用了。如果想要给 DOM 元素添加事件，就直接给它提供一个监听函数（ listener，我们这里，listener 和 handler 就是一个东西 ）即可。

当采用 ES6 class 定义组件的时候，常见的做法就是用 class 的方法作为事件处理函数（ handler )。例如，下面的 `Toggle` 组件渲染了一个按钮，可以让用户在 ON 和 OFF 直接切换：

```js
class Toggle extends React.Component {
  constructor(props) {
    super(props)
    this.state = { isToggleOn: true }

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }))
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    )
  }
}

ReactDOM.render(<Toggle />, document.getElementById('root'))
```

必须谨慎对待 JSX 回调函数中的 this，类的方法默认是不会绑定 this 的。如果你忘记绑定 this.handleClick 并把它传入 onClick, 当你调用这个函数的时候 this 的值会是 undefined 。

这个特点不是 React 特有的，JS 的函数就是这样工作的。一般来讲，如果你用了一个方法，但是没有用 `()` ，例如 `onClick={this.handleClick}` ，就应该考虑绑定（ bind ）这个方法。

如果每次都执行 `bind` 你觉得麻烦，有两种方式可以绕开这个问题。可以用目前还处于试验阶段的 [public class 字段语法](public class fields syntax)，下面的方式就可以直接实现绑定了。

```js
class LoggingButton extends React.Component {
  //这样的语法可以保证 `this` 被绑定进 handleClick
  // 这是一个处于试验阶段的语法
  handleClick = () => {
    console.log('this is:', this)
  }

  render() {
    return <button onClick={this.handleClick}>Click me</button>
  }
}
```

实际中，create-react-app 等环境已经默认配置了对这种方式的支持，所以 Peter 平常用的最多的就是这种方式。

另外，可以在回调时使用 ES6 的箭头函数：

```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this)
  }

  render() {
    // 这种语法也可以保证 `this` 成功绑定进 handleClick
    return <button onClick={e => this.handleClick(e)}>Click me</button>
  }
}
```

这个第二种方式其实就有弊端了，因为每次 `LoggingButton` 渲染，都会创建一次这个函数，会造成性能问题。

## 给事件处理函数传参

开发中常见的一种情况是：有一个循环，每次迭代我们都需要给事件处理函数传递参数。举个例子，如果 `id` 是行 ID ，那么下面两种形式都可以：

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上面两种方式是等价的，一个用了箭头函数，另外一个用了 `Function.prototype.bind` 。

任何一种方式下，代表 React 事件的 `e` 参数都会作为 ID 后面的第二个参数传递给处理函数。用箭头函数的方式，我们直接就名文传递了，用 `bind` 的时候，是自动传递的。

## 参考

* https://reactjs.org/docs/handling-events.html
