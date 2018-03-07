看看 list 使用技巧。

## JS 处理 list

给一个数字组成的数组，执行 `map` 方法 ，每一个数乘以 2 ，组成的新数组赋值给 `doubled` 并且 log 出来。

```js
const numbers = [1, 2, 3, 4, 5]
const doubled = numbers.map(number => number * 2)
console.log(doubled)
```

React 里面，数组变成 list 也是用同样的方式。

## 渲染多个组件

可以把一组元素放到大括号中显示出来。

下面用 map 把数组转换成了一组元素。

```js
const numbers = [1, 2, 3, 4, 5]
const listItems = numbers.map(number => <li>{number}</li>)
```

把整个 `listItems` 数组包含在 `<ul>` 元素中：

```js
ReactDOM.render(<ul>{listItems}</ul>, document.getElementById('root'))
```

代码最终会显示一个列表出来。

## 基本 list 组件

We can refactor the previous example into a component that accepts an array of numbers and outputs an unordered list of elements.

可以把之前的例子重构到一个组件中，组件接受一个数组作为属性，输出一个 list 。

```js
function NumberList(props) {
  const numbers = props.numbers
  const listItems = numbers.map(number => <li>{number}</li>)
  return <ul>{listItems}</ul>
}

const numbers = [1, 2, 3, 4, 5]
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
)
```

代码运行，会看到有警告信息，说应该提供 `key` 。`key` 是一个特殊的字符串属性，创建 list 列表的时候是必须添加的：

```js
function NumberList(props) {
  const numbers = props.numbers
  const listItems = numbers.map(number => (
    <li key={number.toString()}>{number}</li>
  ))
  return <ul>{listItems}</ul>
}

const numbers = [1, 2, 3, 4, 5]
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
)
```

## Key

`key` 的作用是帮助 react 定位到底哪个列表项目被修改了，或者添加删除了。

```js
const numbers = [1, 2, 3, 4, 5]
const listItems = numbers.map(number => (
  <li key={number.toString()}>{number}</li>
))
```

`key` 的字符串要有独一无二性，以便把列表项各个都区分开。

使用数据 id 作为 key 是非常常见的：

```js
const todoItems = todos.map(todo => <li key={todo.id}>{todo.text}</li>)
```

如果实在没有 id 可以用，也可以勉强使用数组索引值（ index ）：

```js
const todoItems = todos.map((todo, index) => (
  // Only do this if items have no stable IDs
  <li key={index}>{todo.text}</li>
))
```

这种方式是不推荐的，可能会有各种不良副作用。

## 参考

* https://reactjs.org/docs/lists-and-keys.html
