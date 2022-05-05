## `setState` 是如何知道要更新DOM的



当你在组件中调用`setState` 的时候，你认为发生了什么

``` jsx
import React from 'react'
import ReactDOM from 'react-dom'

class Button extends React.Component {
  constructor(props) {
    super(props)
    this.state = {clicked: true}
    this.handleClick = this.handleClick.bind(this)
  }
  
  handleClick() {
    this.setState({clicked: true})
  }
  
  render() {
    if(this.state.clicked) {
      return <h1>Thanks</h1>
    }
    
    return(
    	<button onClick={this.handleClick}>
      	Click me!
      </button>
    )
  }
}

ReactDOM.render(<Button />, document.getElementBuId('container'))
```

当然是 React 根据下一个状态 `{clicked: true}` 重新渲染组件， 同时更新 DOM 以匹配返回的 `<h1>Thanks</h1>` 元素

看起来很直白，但是等等， 是 React 做了这些吗 ， 还是 React DOM ？

更新DOM 听起来像是 React DOM  的职责所在， 但是我们调用的是 `this.setState()` , 而没有调用任何来自 React DOM 的东西， 而且我们组件的父类 `React.Component` 也是 React 本身定义的

**所以 存在于 `React.Component` 内部的 `setState()` 是如何 更新 DOM 的呢？**

或许 我们会认为： `React.Component` 类包含了 DOM 更新 的逻辑

但是如果是这样的话, `this.setState()` 如何能在其他环境下使用呢 ， 举个例子 `React Native App ` 中的组件也继承自 `React.Component` 他们依然可以像我们在上面那样调用 `this.setState()` 而且  React Native 渲染的是安卓和IOS 原生界面 而不是 DOM

你或许对 `React Test Render` 或是 `Shallow Renderer` 很熟悉， 这些测试策略能让你正常渲染组件, 也可以 在组件内部调用 `this.setState()` 但是这两个渲染器并不与 DOM 相关

如果你曾使用过一些渲染器像 `React ART `  你也许知道在一个页面中 ， 我们是可以使用多个渲染器（举个例子， ART 组件在 React DOM 树的内部起作用）这使得全局标志或变量无法维持

因此, `React.Component` 以某种未知的方式将处理状态 (state) 更新的任务委托给了特定的平台的代码，在我们理解这些事如何发生的之前， 让我们深挖一下 包是如何分离的以及为什么要这样分离

有一个很常见的误解就是 React “引擎” 是存在于 `React` 包里的， 然而事实并非如此

实际上， 从 `React 0.14` 我们将代码拆分成多个包以来， `react` 包故意只暴露一些特定组件的API, 绝大多数 React 的实现都存在于 "渲染器 renderers" 中

`React-dom`, `react-native`,`react-test-renderer` 都是常见的渲染器

这就是为什么 不管你的平台代码是什么， `react` 包 都是可用的， 从 `react` 包中导出的一切， 比如 ： `React.Component`, `React.createElement`, `React.Children` 和 最终的 `Hooks`, 都是独立于目标平台的， 不论你是运行 `React DOM `还是 `React Native` 你的组件都可以使用同样的方式导入和使用

相比之下 ， 渲染器包暴露的都是特定平台的API， 比如说 ： `ReactDOM.render()`, 可以让你将React 层次结构挂载进一个DOM 节点，每一种渲染器都提供了类似的API， 理想情况下，绝大多数组件都不应该从渲染器中导入任何东西， 只有这样  组件才会更加灵活

和大多数人想在想的一样，React 引擎 就是存在于各个渲染器的内部, 很多渲染器包含一份同样代码的复制，-- 我们称为 "协调器（reconciler）"，构架步骤（build step） 将协调器代码和渲染器代码平滑地整合成一个高度优化的捆绑包（bundle）以获得更高的性能（代码复制通常来说不利于控制捆绑包的大小， 但是绝大多数React 用户同一时间只会选择一个渲染器，比如说 `react-dom`）

这里需要注意的是， `react` 包仅仅是让你使用 React 特性， 但是它完全不知道 这些特性是如何实现的，而渲染器包（`react-dom`, `react-native`）等提供了 React 特性 的实现以及平台特定的逻辑，这其中的有些代码是共享的（协调器）, 但是这就涉及到 各个渲染器的实现细节了

现在我们知道 为什么当我们想使用 新特性时， `react` 和 `react-dom` 都需要被更新，比如 React 16.3 添加了`Context API` `React.createContext()` API 会被 React 包暴露出来

但是 `React.createContext()` 其实并没有实现 context, 因为 在 React DOM  和 REACT DOM  Server ， 中同样一个 API 应当有不同的实现， 所以`createContext()` 只返回了 一些普通的对象

``` jsx
// 简化版代码
function createContext(defaultValue) {
  let context = {
    _currentValue: defaultValue,
    Provider: null,
    Consumer: null
  }
  context.Provider = {
    $$typeof: Symbol.for('react.provider'),
    _context: context
  }
  context.Consumer = {
    $$typeof: Symbol.for('react.context'),
    _context: context
  }
  return context
}
```

当你在代码中使用`<MyContext.Provider>` 或 `<MyContext.Consumer>` 的时候，是渲染器决定如何处理这些接口，React DOM 也许用某种方式追踪context 的值

所以 如果你将react 升级到了 16.3+ ， 但是不更新 `react-dom` , 那么你就使用了一个上不知道 `Provider` 和 `Consumer` 类型的渲染器， 这就是为什么一个老版本的 `react-do` 会报错说这些类型是无效的

同样的 警告也会出现在 react native 中， 然而不同于React DOM  的是，一个 React 新版本的发布 并不会立即强制发布新的React Native 版本， 他们有独立的发布日程，隔一段时间， 更新后的渲染器代码就会单独同步到 react native 仓库中

现在我们知道了 react 包 并不包含任何有趣的东西，具体的实现是存在于 `react-dom`， `react-native` 之类的渲染器中，但是这样并没有回答我们的问题 `React.Component` 中的 `setState()` 如何与正确的渲染器“对话”

答案是 ， 每个渲染器都在已创建的类上设置了一个特殊字段， 这个字段叫做 `updater`  这并不是你要设置的东西，而是 `react DOM`， `React Native` 在创建完你爹类的实例后会立即设置的东西

``` jsx
// React DOM 内部
const inst = new YourComponent()
inst.props = props
inst.updater = ReactDOMUpdater

// React DOM Server 内部
const inst = new YourComponent()
inst.props = props
inst.updater = ReactDOMServerUpdater

// React Native 内部
const inst = new YourComponent()
inst.props = props
inst.updater = ReactNativeUpdater
```

查看React.Component 中 setState 的实现， setState 所做的一切就是 委托渲染器创建这个组件的实例

``` jsx
// 适当简化的代码
setState(partialState, callback) {
  // 使用 updater 字段回应渲染器
  this.updater.enqueueSetState(this, partialState, callback)
}
```

这就是 `this.setState() ` 尽管定义在React 包中，确能够更新 DOM 的原因， 它读取由 React DOM 设置的 `this.updater` ,  让 React DOM 安排并处理更新

现在关于累的部分我们已经知道了， 那关于 Hooks 的呢

Hooks 使用了一个 “dispatcher” 对象， 代替了 updater 字段， 当你调用 `React.useState()` ， `React.useEffect()` 或者其他内置的 Hook 时， 这些调用被转发给了当前的dispatcher 

``` jsx
// React 内部 （简化）
const React = {
  // 真实属性比较深
  _currentDispatcher: null,
  
  useState(initialState) {
    return React._currentDispatcher.useState(initialState)
  }
  
  useEffect(initialState) {
    return React._currentDispatcher.useEffect(initialState)
  }
  // ...
}
```

各个渲染器都会在渲染你的组件之前，设置 dispatcher

``` jsx
// React DOM 内部
const prevDispatcher = React._currentDispatcher
React._currentDispatcher = ReactDOMDispatcher
let result
try{
  result = YourComponent(props)
} finally {
  // 恢复原状
  React._currentDispatcher = prevDispatcher
}
```

`updater` 字段和 `_currentDispatcher` 对象都是称为依赖注入的通用编程原则的形式，这两种情况下，渲染器将诸如 `setState` 之类的功能的实现 注入到 通用的 React 包中，以使组件更具声明性