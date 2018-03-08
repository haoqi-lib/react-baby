很多时候，多个组件需要体现出同一个数据的修改。这时候可以用 redux ，也可以用 **state 提升** ，把这个数据提升到所有相关组件的共同父组件中。

## 温度计算器

来创建一个温度计算器，给定一个温度，它可以运算出水是否会沸腾。

我们先创建一个叫 `BoilingVerdict` （沸腾裁断）的组件：

```js
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>
  }
  return <p>The water would not boil.</p>
}
```

输入的摄氏度如果大于一百，那么这个组件就会返回沸腾，否则返回没沸腾。

接下来，咱们创建一个 `Calculator` 组件，里面有接收用户输入温度用的 `<input>` ，输入值会保存到 `this.state.temperature` 。

另外也显示了 `BoilingVerdict` 组件。

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
    this.state = { temperature: '' }
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value })
  }

  render() {
    const temperature = this.state.temperature
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input value={temperature} onChange={this.handleChange} />

        <BoilingVerdict celsius={parseFloat(temperature)} />
      </fieldset>
    )
  }
}
```

## 添加第二个 Input

新需求是这样，需要添加一个可以添加华氏度的 input ，它要和摄氏度的 input 保持同步。

先来抽出 `TemperatureInput` 组件。会有一个 `scale` 属性，值可能是 `c` 或者 `f` 。

```js
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
}

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
    this.state = { temperature: '' }
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value })
  }

  render() {
    const temperature = this.state.temperature
    const scale = this.props.scale
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    )
  }
}
```

现在修改 Calculator 来渲染两个独立的 input ：

```js
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    )
  }
}
```

现在我在一个 input 输入温度，另一个是不会自动更新的。

同时，我们也不能显示 `BoilingVerdict` ，现在 `Calculator` 并不知道当前温度是多少，因为温度值被隐藏在 `TemperatureInput` 内部了。

## 添加转换函数

来写两个函数，把摄氏变华氏，把华氏变摄氏：

```js
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9
}

function toFahrenheit(celsius) {
  return celsius * 9 / 5 + 32
}
```

这两个函数都是转换数字的。再来写一个函数接收字符串型的温度和转换函数作为参数，返回一个字符串。我们用它来根据一个 input 值计算出另一个 input 值。

如果输入温度不合法，它会返还空字符串。它会把输出精度保留到小数点后面第三位：

```js
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature)
  if (Number.isNaN(input)) {
    return ''
  }
  const output = convert(input)
  const rounded = Math.round(output * 1000) / 1000
  return rounded.toString()
}
```

例如 `tryConvert('abc', toCelsius)` 返回一个空字符串，`tryConvert('10.22', toFahrenheit)` 返回 `50.396` 。

## state 提升

两个 `TemperatureInput` 组件目前都是把自己的温度保存在本地 state 中：

```js
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
```

但是，我们希望的效果是这两个温度能够自动同步。当我更新摄氏度输入的时候，华氏度那个 input 应该显示出转换出的文档值，反之亦然。

React 条件下，共享 state 是通过把它移动到各个需要这个数据的组件的最近的共同父组件中。这就是所谓的 state 提升。我们来吧本地 state 从 `TemperatureInput` 提升到 `Calculator` 。

It can instruct them both to have values that are consistent with each other. Since the props of both components are coming from the same parent Calculator component, the two inputs will always be in sync.

当 `Calculator` 获得了要被共享的 state ，它就成了两个 input 的当前温度值的唯一数据源。两个 `TemperatureInput` 组件都从父组件 `Calculator` 中获得温度值，所以同步就不再是问题了。

来看如何一步步的实现这个效果。

首先，咱们把 `TemperatureInput` 组件中的 `this.state.temperature` 改成 `this.props.temperature` 。咱们暂时假定 `this.props.temperature` 是存在的。

```js
render() {
  // Before: const temperature = this.state.temperature;
  const temperature = this.props.temperature;
  // ...
```

属性是只读的。当温度保存在 state 中时，`TemperatureInput` 可以直接用 `this.setState()` 来修改它。但是现在 `TemperatureInput` 不能控制它了。

React 中，这个问题的解决方法通常就是把组件变成受控的。就像 `<input>` 可以使用 `value` 和 `onChange` 属性一样，`TemperatureInput` 可以通过使用 `temperature` 和 `onTemperatureChange` 这两个属性，从 `Calculator` 中接收信息。

现在，当 `TemperatureInput` 需要更新自己的温度时，就可以呼叫 `this.props.onTemperatureChange` ：

```js
handleChange(e) {
  // Before: this.setState({temperature: e.target.value});
  this.props.onTemperatureChange(e.target.value);
  // ...
```

`temperature` 和 `onTemperatureChange`  并没有什么特殊含义，叫什么名字都可以。

`onTemperatureChange` 和 `temperature` 都会由 `Calculator` 提供。输入有修改时，`Calculator` 执行自己的函数，修改自己的 state 值。从而两个 input 也都会跟着更新。

在修改 `Calculator` 之前，来回顾一下 `TemperatureInput` 的改动。本地的 state 被移除了，取而代之的是 `this.props.temperature` 。修改温度也不能再用 `setState` 了，需要用 `this.props.onTemperatureChange()` 。

```js
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props)
    this.handleChange = this.handleChange.bind(this)
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value)
  }

  render() {
    const temperature = this.props.temperature
    const scale = this.props.scale
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    )
  }
}
```

现在来改 `Calculator` 。

我们会把当前 input 的温度和单位保存到本地 state 中。这个 state 就是从 input 组件中提升上来的，会作为 input 值的唯一数据源。它是一个最简数据，但是由它可以  推算得到两个 input 的值。

例如，如果在  设置度的 input 中输入 37 ，那么 `Calculator` 的这个 state 值就是：

```js
{
  temperature: '37',
  scale: 'c'
}
```

如果稍后编辑华氏度的 input ，填写 212 ，那么此时 state 就是：

```js
{
  temperature: '212',
  scale: 'f'
}
```

我们可以把两个 input 中的值都存起来，但是实际上没有必要。只要把最新被修改的那个 input 的值保存即可。还有就是它的  单位。这样，另一个 input 值可以由这个值运算出来。

两个 Input 可以保持同步，因为它们的数据的源都是一个 state ：

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props)
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this)
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this)
    this.state = { temperature: '', scale: 'c' }
  }

  handleCelsiusChange(temperature) {
    this.setState({ scale: 'c', temperature })
  }

  handleFahrenheitChange(temperature) {
    this.setState({ scale: 'f', temperature })
  }

  render() {
    const scale = this.state.scale
    const temperature = this.state.temperature
    const celsius =
      scale === 'f' ? tryConvert(temperature, toCelsius) : temperature
    const fahrenheit =
      scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange}
        />

        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange}
        />

        <BoilingVerdict celsius={parseFloat(celsius)} />
      </div>
    )
  }
}
```

Now, no matter which input you edit, and in the Calculator get updated. One of the inputs gets the value as is, so any user input is preserved, and the other input value is always recalculated based on it.

现在不管咱们修改那个 input ，`this.state.temperature` 和 `this.state.scale` 都会被更新。一个 input 会值保存不变，另一个会重新运算保持跟另一个 input 同步。

看一下如果修改 input 都会发生哪些步骤：

* React 会执行 `TemperatureInput` 组件中 `handleChange` 。
* `this.props.onTemperatureChange()` 于是会被执行，它是父组件 `Calculator` 提供的 。
* 取决于到底是那个 input 被编辑，`handleFahrenheitChange` 或者 `handleCelsiusChange` 会被执行
* 这些方法中会呼叫 `setState` 来更新 state ，同时也会造成界面的重新渲染。
* React 执行 `Calculator` 组件的 `render` 方法。
* React 会执行 `TemperatureInput` 的 `render` 。
* React DOM 会更新 DOM 显示出最新的 input 值。

每一次更新都会重复相同的步骤，所以 input 的内容始终会保存同步。

## 参考

* https://reactjs.org/docs/lifting-state-up.html
