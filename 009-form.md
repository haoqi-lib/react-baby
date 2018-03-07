聊聊 React 中表单的使用，主要涉及受控组件（ controlled component ）的概念。

## form 自带 state

form 这个元素在 React 这里非常特别，因为它天然的就拥有自己的内部 state 。比如下面的 HTML form 接收 name 。

```js
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

form 的原本工作方式就是，提交后会跳转到其他页面，在 React 条件下这个特性也一样工作。但是大多数情况下 React 下用 form 有自己的一套思路。例如，会专门定义一个函数来处理表单提交等等，这就涉及到了受控组件的概念。

## 受控组件

普通 HTML 条件下， `<input>` ，`<textarea>` 和 `<select>` 这些组件都有自己的 state ，会根据用户输入而改变。React 的原则是，一切会被修改的数据都应该保存为 state ，只能通过 setState 来修改。

这两种思路要融合到一起，就需要把 React 的 state 作为唯一数据源。用户有输入，数据也要先交给 React ，或者说被 React 控制，所以这样的组件叫做受控组件。

之前的例子要是写成受控的，就会是下面这样：

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props)
    this.state = { value: '' }

    this.handleChange = this.handleChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleChange(event) {
    this.setState({ value: event.target.value })
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value)
    event.preventDefault()
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input
            type="text"
            value={this.state.value}
            onChange={this.handleChange}
          />
        </label>
        <input type="submit" value="Submit" />
      </form>
    )
  }
}
```

form 元素上一旦设置了 `value` 属性，那么显示的内容就会一直是 `this.state.value` 。这样，React state 就是唯一数据源了。每次用户敲一个键，`handleChange` 都会执行一次，更新 state 值。所以显示的内容也会随着用户输入而改变。

对于受控组件，每一次修改 state 都会触发对应的相应函数，这个其实可以带来很多方便，比如可以在函数中把用户输入都变成大写：

```js
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

## textarea 标签

HTML 条件下，`<textarea>`通过 children 来存放要显示的内容：

```js
<textarea>Hello there, this is some text in a text area</textarea>
```

而在 React 条件下，`<textarea>` 用一个 `value` 属性来存放要显示的内容，所以用起来跟 `<input>` 就没有太多区别了：

```js
class EssayForm extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    }

    this.handleChange = this.handleChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleChange(event) {
    this.setState({ value: event.target.value })
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value)
    event.preventDefault()
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    )
  }
}
```

`this.state.value` 在构造函数中赋了初始值，所以一开始就会显示一些默认内容。

## select 标签

In HTML, <select> creates a drop-down list. For example, this HTML creates a drop-down list of flavors:

HTML 中， `<select>` 创建一个下拉列表。例如下面的 html 生成了一个下拉的风味列表：

```js
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">
    Coconut
  </option>
  <option value="mango">Mango</option>
</select>
```

`Coconut` 这一项是默认选中的。选中是用 `selected` 属性体现的。React 中，不用这个属性，而是用 select 标签上的 `value` 属性：

```js
class FlavorForm extends React.Component {
  constructor(props) {
    super(props)
    this.state = { value: 'coconut' }

    this.handleChange = this.handleChange.bind(this)
    this.handleSubmit = this.handleSubmit.bind(this)
  }

  handleChange(event) {
    this.setState({ value: event.target.value })
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value)
    event.preventDefault()
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite La Croix flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    )
  }
}
```

可以看到 `<input type="text">`, `<textarea>`, 和 `<select>` 的工作方式都很类似，都是通过 `value` 来进行控制.

如果要选中多个项目，可以传递数组作为 `value` 值：

```js
<select multiple={true} value={['B', 'C']}>
```

## file input 标签

HTML 中，`<input type="file">`让用户可以从硬盘上选择文件：

```js
<input type="file" />
```

因为值是只读的，所以 React 中它是一个[**非**受控组件](https://reactjs.org/docs/uncontrolled-components.html#the-file-input-tag)。

## 处理多个输入

当需要处理多个受控 `input` 的时候，可以给每个 input 添加一个 `name` 属性，然后就可以在响应函数中通过 `event.target.name` 来灵活处理了。

例如：

```js
class Reservation extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    }

    this.handleInputChange = this.handleInputChange.bind(this)
  }

  handleInputChange(event) {
    const target = event.target
    const value = target.type === 'checkbox' ? target.checked : target.value
    const name = target.name

    this.setState({
      [name]: value
    })
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange}
          />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange}
          />
        </label>
      </form>
    )
  }
}
```

我们使用了 ES6 的**运算属性名**语法，来根据 input 的 name 更新对应 key 值的 state ：

```js
this.setState({
  [name]: value
})
```

等价于下面的 ES5 写法：

```js
var partialState = {}
partialState[name] = value
this.setState(partialState)
```

并且，因为 `setState` 会自动把部分 state 融合进当前 state ，所以我们只需要关心修改的部分即可。

## 受控 Input 中的 Null Value

设置了 `value` 属性的受控组件，用户是不能直接编辑的。但是如果设置了 `value` ，但是 `input` 还是可以直接编辑的，那么很可能就是不小心把 value 设置成了 `undefined` 或者 `null` 。

下面的例子展示了这个现象（ input 首先被锁住，然后就可以被编辑了）：

```js
ReactDOM.render(<input value="hi" />, mountNode)

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode)
}, 1000)
```

## 受控组件的替代方案

使用受控组件有时候的确会显得太麻烦。一个替代方案是使用 [uncontrolled components](https://reactjs.org/docs/uncontrolled-components.html) 来实现表单。但是推荐的还是受控组件的方式。

## 参考

* https://reactjs.org/docs/forms.html
