## 为什么我们要写 `super(props)`



首先， `super(props)` 出现在我们代码里的次数比我知道的还要多

``` jsx
class Checkbox extends React.Component{
  constructor(props) {
    super(props)
    this.state = {isOn: true}
  }
}
```

当然 也可以通过 `class fields proposal` 来省略这个声明

``` jsx
class Checkbox extends React.Component {
  state = {isOn: true}
}
```

早在 2015 年 React 0.13 已经计划支持, 在当时，声明 `constructor` 和 `super(props)` 一直被视作暂时的解决方案, 直到有合适的类字段声明形式

但在此之前， 我们先回到 ES2015风格的代码

``` jsx
class Checkbox extend React.Compnent {
  constructor(props) {
    super(props)
    this.state = {isOn: true}
  }
}
```

 为什么我们要调用`super` ，可以不这么做吗， 如果在调用时 不传入 `props` 又会发生了什么

在 `javascript` 中， `super` 指父类的构造函数 （在我们的例子中， 他指向了 `React.Component` 的实现）

值得注意的是， 在调用父类的构造函数之前，你是不能在 `constructor` 中使用`this` 关键字的

``` jsx
class Checkbox extends React.Component {
  constructor(props) {
    // 在这里我们还不能使用this
    super(props)
    // 在这里可以了
    this.state = {isOn: true}
  }
}
```

javascript 有足够合理的动机来强制你在接触 this 之前执行父类构造函数

``` jsx
class Person {
  constructor(name) {
    this.name = name
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues() // 这是禁止的
    super(name)
  }
}
```

是想一下 ， 在调用 `super` 之前使用 `this` 不被禁止的情况下， 一个月后， 我们可能在 `greetColleagues` 打印的消息中使用了  person  的  name 属性

``` jsx
greetColleagues() {
  alert('xxxxxxxxxx')
  alert('my name is ' + this.name)
}
```

此时  this.name 尚未定义

为了避免落入这个陷阱， javascript 强制你在使用 this 之前强行调用 super 让父类来完成这个事情

``` jsx
constructor(props) {
  super(props)
}
```

这里留下了另一个问题： 为什么要传入 props

你或许会想到，为了让 React.Component 构造函数能够初始化`this.props`, 将`props`传入 `super` 是必须的

``` jsx
// React 内部
class Component {
  constructor(props) {
    this.props = props
  }
}
```

但有些疑惑的是， 即便你调用 `super()` 没有传入 `props` 你依然能够在 `render` 函数或其他方法中访问到`this.props`

那么这是怎么做到的，实际上， React 在调用构造函数后也立即将`props`赋值到了实例上

``` jsx
// React 内部
const instance = new YourComponent(props)
instance.props = props
```

因此即便你忘记了将`props` 传给 `super`，React 也会在之后将它定义到实例上



这意味着你能够用 `super()` 代替 `super(props)` 吗

最好不要，毕竟这样写在逻辑上并不明确，React 会在构造函数执行完毕后给this.props赋值，但是会 this.props 在 super 调用 知道 构造函数结束期间值为undefined

``` jsx
// React 内部
class Component {
  constructor(props) {
    this.props = props
  }
}

// 你的程序内部
class Button extends React.Component {
  constructor(props) {
    super() // 此时传入了 undefined
    console.log(props) // {}
    console.log(this.props) // 未定义
  }
}
```

所以这也是建议开发者执行`super(props)` 的原因， 即使理论上这并非必要

``` jsx
class Button extends React.Component {
  constructor(props) {
    super(props)
    console.log(props) // {}
    console.log(this.props) // {}
  }
}
```

这样就确保了 `this.props` 在构造函数执行完毕之前就已经赋值