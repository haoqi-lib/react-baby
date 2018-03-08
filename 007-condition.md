有时候需要根据应用状态，有选择的渲染部分组件。

## 条件渲染

条件渲染，就是根据不同条件决定渲染哪部分组件。React 的条件渲染跟 JS 中的条件的工作方式是完全一样的。就是用 JS 的 if 或者三目运算符来完成。

例如有下面两个组件：

```js
function UserGreeting(props) {
  return <h1>Welcome back!</h1>
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>
}
```

下面来创建一个 `Greeting` 组件，来根据用户是否已经登录，来显示不同的打招呼信息：

```js
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn
  if (isLoggedIn) {
    return <UserGreeting />
  }
  return <GuestGreeting />
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
)
```

这个例子中，随着 `isLoggedIn` 的值的不同，会显示出不同的内容。

## 元素变量

可以用变量来存储元素。用上这种元素变量，可以让我们根据条件渲染部分内容，而保持其他内容不变，说起来抽象，下面例子一看便知。

下面又两个组件分别是 Logout 和 Login 按钮：

```js
function LoginButton(props) {
  return <button onClick={props.onClick}>Login</button>
}

function LogoutButton(props) {
  return <button onClick={props.onClick}>Logout</button>
}
```

下面我们要创建一个组件叫 `LoginControl` 。它会根据自己的当前状态决定渲染 `<LoginButton />` 还是 `<LogoutButton />` 。同时也会渲染 `<Greeting />` ：

```js
class LoginControl extends React.Component {
  constructor(props) {
    super(props)
    this.handleLoginClick = this.handleLoginClick.bind(this)
    this.handleLogoutClick = this.handleLogoutClick.bind(this)
    this.state = { isLoggedIn: false }
  }

  handleLoginClick() {
    this.setState({ isLoggedIn: true })
  }

  handleLogoutClick() {
    this.setState({ isLoggedIn: false })
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn

    let button = null
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    )
  }
}

ReactDOM.render(<LoginControl />, document.getElementById('root'))
```

`if` 这种方式比较强大，但是写的太复杂，下面看看有没有在行内解决条件判断的方式。

## 通过 && 元素符实现行内 if

You may embed any expressions in JSX by wrapping them in curly braces. This includes the JavaScript logical && operator. It can be handy for conditionally including an element:

JSX 中可以嵌入任意的表达式，用大括号括起来即可。所以当然也可以使用 `&&` 了：

```js
function Mailbox(props) {
  const unreadMessages = props.unreadMessages
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 && (
        <h2>You have {unreadMessages.length} unread messages.</h2>
      )}
    </div>
  )
}

const messages = ['React', 'Re: React', 'Re:Re: React']
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
)
```

JS 中，`true && expression` 一定求值为 `expression` ，而 `false && expression` 求值结果一定为 `false` 。

这样，如果条件为 true ，那么 `&&` 后面的元素就会显示出来，如果条件为 false ，那么这些内容就会被跳过。

## 通过三目运算符实现行内 if-else

`condition ? true : false` 这种三目运算符（ ternary operator ），或者更官方的说法叫条件运算符（ conditional operator )，也是在 JSX 中很常用的。

看下面的例子：

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

条件不同，显示不同的字符串。

当然问好后面也可以跟元素：

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```

具体用那种风格，主要就是照顾代码可读性了。

## 阻止组件渲染

如果根据某种条件，不想让组件显示，`return null` 即可。

```js
function WarningBanner(props) {
  if (!props.warn) {
    return null
  }

  return <div className="warning">Warning!</div>
}

class Page extends React.Component {
  constructor(props) {
    super(props)
    this.state = { showWarning: true }
    this.handleToggleClick = this.handleToggleClick.bind(this)
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }))
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    )
  }
}

ReactDOM.render(<Page />, document.getElementById('root'))
```

组件最后返回 null ，不会影响生命周期函数的运行。

## 参考

* https://reactjs.org/docs/conditional-rendering.html
