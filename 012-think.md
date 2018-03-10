React 真的是做大型 Web 应用的利器。React 的一个特别大的好处就是它能引导你正确的去思考如何组织大型应用。本节来一起做一个可搜索的商品列表，实践一下 React 的思路。

## 基于一个原型图开始动手

假设现在咱们已经有了 JSON API 和一个如下的原型图：

![](002-mock.png)

JSON API 返回的数据是：

```
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## 第一步：UI 拆分成各级组件

首先就是在设计图上通过画方框的形式拆分出各个组件和子组件，然后每个组件都起个名字。设计师如果细心的化，可能拿到到手的 photoshop 的层名，或者 sketch 的组名，就已经是很好的组件名了。

但是如何去确定哪些内容适合组成一个组件呢？方法跟组织函数和对象的思路一样。可以遵循“单一职责原则”，也就是一个组件就只做一件事。如果这件事变得越来越复杂，那就再往下拆子组件出来。

事件上，如果数据模型本身建立的很好，你会发现拆分组件就变得非常的简单，因为组件和数据之间会形成非常简单的一一对应关系。

![](003-comp.png)

咱们的小应用可以拆分成五个组件：

* FilterableProductTable （黄色）整个应用
* SearchBar （蓝色）接收用户输入
* ProductTable （绿色） 展示数据
* ProductCategoryRow （蓝绿色） 每个类别的标题
* ProductRow （红色） 展示一行数据

`ProductTable` 的头部文件我们并没有单独抽出成一个组件，因为比较简单，如果将来内容多了，也可以抽出成 `ProductTableHeader` 组件。

现在来根据图组织一下组件的层级结构：

* FilterableProductTable
  * SearchBar
  * ProductTable
    * ProductCategoryRow
    * ProductRow

## 第二步：用 React 做一个静态版本

有了层级结构，现在就可以动手实现了。最简单的方式就是先做 UI ，不做交互。做一下步骤分解很有必要，因为做静态版本主要就是打字，不用太动脑子。而做交互就要多动脑子，打字倒是不多。

要做出一个能够展示数据的静态版本，需要做一些组件，里面再包含一些子组件，父子组件间通过 props 传递数据。这个阶段，不需要使用 state ，因为 state 专门是负责交互的。state 的本意就是一些随着时间而发生变化的数据。那既然是静态版本，所以肯定是不用的。

开发过程既可以自上而下，也可以反之。也就是说，可以先做最顶端的组件，例如 `FilterableProductTable` ，或者也可以先做底端的，例如 `ProductRow` 。比较小的应用中，自上而下更简单，但是大型应用中，自下而上，边写组件边写测试是更好的策略。

这一步结束，就可以拥有一些可以复用的组件了。这些组件都只是有一个 `render` 方法，因为是静态的。顶级组件 `FilterableProductTable` 接收数据模型为自己的 prop 。可以试着修改一下数据，看看 UI 是否能随着改变。React 遵循“单向数据流”（或者叫做单向绑定），所以根本结构是非常简单的。

## 确定 state 结构

目标：最简，完备。

要让 UI 交互起来，我们需要有修改数据的能力，React 通过 state 来完成这个工作。

第一步要做的就是抽象出应用需要的可修改 state 值的最小集合。关键就是要注意 DRY （ Don't Repeat Yourself ），也就是不重复。保留最简单的数据，其他数据通过演算得到。例如，如果做一个 Todo 应用，那就只保存 todos 数组即可，至于全部 todo 数量 ，完全可以运算得到，无需专门保存到 state 中。

我们自己的应用的数据如下：

* 原始商品列表
* 用户输入
* checkbox 的值
* 筛选后的商品列表

来看看那些应该作为 state 值出现，其实就是要问下面三个问题：

* 它是否是从父元素通过 props 传递过来的？如果是，那它基本上就不应该是 state 。
* 它是否会变化？如果不变，当然就不是 state 。
* 它是否可以从其他的数据演算得到？如果是，就不该作为 state 。

原始商品列表是通过属性传入的，所以它不是 state 。如果输入的搜索关键字和 checkbox 值，应该是 state ，因为它们会随时变化，也不能从其他数据演算得到。最后，筛选后的商品列表不应该是 state ，因为它可以通过把原始列表和用户输入进行混合运算而得到。

所以，咱们的 state 有：

* 用户输入的搜索关键字
* checkbox 的值

## 第四步：确定 state 应该从属哪个组件

state 定下之后，要来确定 state 从属于哪个组件，会被哪个组件修改。

记住，React 的核心思路就是单向数据流，数据从上往下流动。state 到底应该属于哪个组件对新手来说是有挑战性的，所以咱们来按照下面的步骤操作：

拿出一个 state 值

* 找出所有要基于它进行渲染的组件
* 找到这些组件的共同父组件
* 真正拥有这个 state 值的，应该是这个父组件，或者父组件的之上的组件
* 如果咱们找不到一个合理的组件保存这个 state ，那就专门创建一个高于共同父组件的组件来存放 state

在咱们的例子里实践一下这个思路：

* `ProductTable` 和 `SearchBar` 都需要咱们的 state 值
* 它们的共同父组件是 `FilterableProductTable`
* 把搜索关键字和 ckeckbox 值存放到 `FilterableProductTable` 组件中，概念上也是合适的

所以，最终可以决定，state 值就放到 `FilterableProductTable` 组件里即可。首先，给 `FilterableProductTable` 组件添加 `state = {filterText: '', inStockOnly: false}` 属性。然后，把 `filterText` 和 `inStockOnly` 作为属性传递给 `ProductTable` 和 `SearchBar` 。最后，使用这些数据来筛选商品和设置 `SearchBar` 显示。

现在，设置 `filterText` 为 `"ball"` ，刷新一下 app 。可以看到商品列表成功的被筛选了。

## 第五步：添加反向数据流

现在咱们的应用数据都是从上到下单向流动的。现在咱们要支持反方向的数据流了： form 所在组件是比较底层的组件了，我们需要它去修改 `FilterableProductTable` 组件的状态值。

React 让这个反向数据流动过程非常明显，这样可以让咱们更好的理解应用工作原理，当然这样比传统的双向数据绑定的形式要多打一些字。

现在如果输入搜索关键字，会发现应用会完全忽略咱们的输入，这个是有意为之的。`input` 的值是由从 `FilterableProductTable` 传递过来的 `state` 控制的。

我们最终想要的效果其实是这样：当用户修改 form 的时候，state 会被更新。又因为组件只能修改自己的 state ，所以 `FilterableProductTable` 需要传递回调函数给 `SearchBar` ，这样 `SearchBar` 通过触发这个函数，就能修改父组件中的值了。

## 总结

React 的思路比传统思路看似麻烦一些，但是实际上达成了很好的模块化，未来随着组件不断被复用，React 程序会呈现出更好的可读性，甚至是更少的代码量。

## 参考

* https://reactjs.org/docs/thinking-in-react.html
