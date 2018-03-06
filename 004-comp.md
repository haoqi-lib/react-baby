组件（ components ）可以让我们把 UI 分割成独立的可以复用的片段。概念上来讲，组件类似于 JS 的函数，它接收任意的输入（也就是 props ，属性），返回 React 元素。

## 函数式和 class 式组件

定义一个组件最简单的方式是写一个 JS 的函数

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>
}
```

这个函数就是一个完整的 React 组件，因为它接收一个 props 对象作为参数，返回一个 React 元素。这样的组件叫做函数式组件，因为它的确就是个函数。

另外一个定义组件的方式就是使用 ES6 的 class

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>
  }
}
```

从 React 的角度，上面两个组件是等价的。

不过 class 式组件功能会多一些。

## 渲染一个组件

前面，我们只是看到了由 html 标签组成的 React 元素

```js
const element = <div />
```

实际上，元素也可以由用户自己定义的组件组成

```js
const element = <Welcome name="Sara" />
```

> 组件名一定要首字母大写

## 组件的组合

组件中可以使用其他组件。

例如可以由一个 App 组件，里面使用很多次 Welcome 组件。

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

## 组件的抽离

可以把大组件抽离成多个小组件。

例如

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img
          className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">{props.author.name}</div>
      </div>
      <div className="Comment-text">{props.text}</div>
      <div className="Comment-date">{formatDate(props.date)}</div>
    </div>
  )
}
```

Comment 这个组件接收 author ， text ， data 这几个属性。

可以抽离出多个组件出来。

首先先抽离 Avatar

```js
function Avatar(props) {
  return (
    <img className="Avatar" src={props.user.avatarUrl} alt={props.user.name} />
  )
}
```

Avatar 不需要关系 Comment 中显示的到底是谁的头像，所以它的属性名就用 user 即可，不用 author 。

推荐的命名属性的方式是：从组件自己的角度出发，而不是从它被使用的上下文出发。

这样， Comment 组件就简单很多了。

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">{props.author.name}</div>
      </div>
      <div className="Comment-text">{props.text}</div>
      <div className="Comment-date">{formatDate(props.date)}</div>
    </div>
  )
}
```

接下来可以抽离出 UserInfo 组件，里面显示 Avater 和 username 。

```js
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">{props.user.name}</div>
    </div>
  )
}
```

这样 Comment 就可以写的更简单了

```js
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">{props.text}</div>
      <div className="Comment-date">{formatDate(props.date)}</div>
    </div>
  )
}
```

抽离组件的总体规则就是：如果一项内容要经常复用，或者说不复用但是足够复杂，都适合抽离成独立组件。

## 属性是只读的

不管是函数式组件还是 class 式组件，都不能去修改自己的属性。

比如这里的 sum 函数

```js
function sum(a, b) {
  return a + b
}
```

这样的函数被叫做纯函数，因为它不会去修改自己的输入，同时给定相同输入一定能返回相同的输出。

而下面这个就是不纯的

```js
function withdraw(account, amount) {
  account.total -= amount
}
```

因为，这个函数试图修改自己的输入。

React 有一条法则

> 组件必须要跟一个纯函数一样，不能修改自己的属性

当然，组件 UI 是动态的，是会变来变去的。这个变化要通过 state 来控制。state 让组件可以根据用户操作，网络返回或者其他的情况来调整自己的输出。有了 state ，纯函数法则就不会被破坏了。
