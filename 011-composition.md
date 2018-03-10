React 有一个很强大的组合模型，官方推荐的方式是使用组合（ composition ）而不是继承（ inheritance ）来复用组件。

下面来展示一些新手倾向与用继承来完成的任务，看看如何来用组合的方式更好的完成。

## 处理包含关系

有些组件事先是不知道自己的 children 是谁的。例如一些”盒子类“的组件，Sidebar 或者 Dialog 这些。

对于这些组件，推荐使用一个特殊的 `children` 属性，把子元素直接传入进来。

```js
const FancyBorder = props => {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  )
}
```

这样，其他用到 `FancyBorder` 的元素就可以通过 JSX 嵌套的方式往它里面传入任意的子元素了。

```js
const WelcomeDialog = () => {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">Welcome</h1>
      <p className="Dialog-message">Thank you for visiting our spacecraft!</p>
    </FancyBorder>
  )
}
```

`FancyBorder` 标签内嵌入和所有 JSX 代码，都会作为 `children` 属性传入 `FancyBorder` 组件。因为 `FancyBorder` 会把 `children` 的内容渲染到一个 `div` 之内，所以被传入的这些元素就都会最终出现在输出中。

另外一种不是那么常见的情况是，一个组件内，我们可能需要多个“洞”，这种情况下咱们可以定一个自己的方式，不用 `children` ：

```js
const SplitPane = props => {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">{props.left}</div>
      <div className="SplitPane-right">{props.right}</div>
    </div>
  )
}

const App = () => {
  return <SplitPane left={<Contacts />} right={<Chat />} />
}
```

React 组件其实就是对象，所以可以跟其他数据一样作为属性值进行传递。

## 特例化

有些组件其实就是另外某个组件的特例。例如 `WelcomeDialog` 就是 `Dialog` 的一个特例。

这个也是可以通过组合来实现的。比较特例化的组件，就是比较通用化的组件中传递一些具体的属性：

```js
const Dialog = props => {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
    </FancyBorder>
  )
}

const WelcomeDialog = () => {
  return (
    <Dialog title="Welcome" message="Thank you for visiting our spacecraft!" />
  )
}
```

对于 class 式组件，组合这种方式一样好用：

```js
const Dialog = props => {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
      {props.children}
    </FancyBorder>
  )
}

class SignUpDialog extends React.Component {
  state = { login: '' }

  render() {
    return (
      <Dialog
        title="Mars Exploration Program"
        message="How should we refer to you?"
      >
        <input value={this.state.login} onChange={this.handleChange} />

        <button onClick={this.handleSignUp}>Sign Me Up!</button>
      </Dialog>
    )
  }

  handleChange = e => {
    this.setState({ login: e.target.value })
  }

  handleSignUp = () => {
    alert(`Welcome aboard, ${this.state.login}!`)
  }
}
```

## 那继承到底还用不用？

以 Facebook 为例，公司里面写了成千上万的组件，从来没有发现必须用继承的情况。

属性和组合让我们非常灵活也非常安全的定制组件的行为和 UI 。不要忘记，组件可以接收的属性类型是没有啥限制的，可以是各种原始值，React 元素或者函数。

如果想要复用的是非 UI 的内容，建议抽出成 JS 模块，然后组件就可以导入模块，使用里面的对象，函数或者类，所以说也用不着使用继承的方式来扩展自身功能。

## 参考

* https://reactjs.org/docs/composition-vs-inheritance.html
