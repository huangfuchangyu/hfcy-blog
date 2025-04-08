## React Hooks : 不是魔法， 仅仅是Array[译]

原文地址： https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e



#### hooks的规则

React 团队规定了2条使用Hooks的主要规则

* 不要在 循环， 条件，或者 循环函数（闭包？）中调用 Hooks
* 只能在 React Function 里调用 Hooks

我认为，第二条是很明显的，想要给 function component 附加一些特性，你需要以某种方法将他们关联起来

然而，对于第一条，我认为这令人感到困惑，因为这样使用API看起来很奇怪，所以，这也是我今天为什么想要解释的



#### Hooks 中的状态管理 都是与数组有关

为了更清晰的理解，让我们看一下一个简单的Hoods 实现起来是什么样的

请注意，这只是推测，而且是只其中一种可能实现的方式，以便于更好的理解它，这不是其内部的真正实现方式



#### 如何实现 `useState()`

通过下面的实例，让我们看一下 Hooks 是怎么工作的

首先，从一个组件开始

``` jsx
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState('Rudi')
  const [lastName, setLastName] = useState('YardLey')
  
  return (
    <Button onClick={() => setFirstName('Fred')}></Button>
  )
}
```

Hooks 背后的思路是，是可以使用一个setter 函数 被 Hooks 以数组的第二个参数返回，并且 这个 setter 函数 能更改状态，它由Hooks 提供



#### So what's React going to do with this

让我们看一下它在React内部是如何工作的，下面将会为我们展示在执行上下文内部是如何 渲染一个特定的组件 ，那意味着这些状态会在将要渲染的组件之外存储。这些状态不是和其他组件共享的，它有一个作用域， that is accessible to subsequent rendering of the specific component

1. 初始阶段

创建两个空数组，一个存放`setter` 另一个存放 `state`

当前 index 为 0

[![bF9wMn.png](https://s4.ax1x.com/2022/02/24/bF9wMn.png)](https://imgtu.com/i/bF9wMn)

2. 第一次渲染

​	第一次执行组件方法

​	调用每一个`useState()`, 当第一次运行，setter function 被 	被 push 到 setter array ，然后， state 被 push 进入 state array

[![bFCH00.png](https://s4.ax1x.com/2022/02/24/bFCH00.png)](https://imgtu.com/i/bFCH00)

3. 接下来的渲染

​	接下来的每次渲染，index 都会被重置，然后从对应的数组中取值



4. 事件处理

​	每个 setter 都有一个引用 指向它的角标位置，当任何一次 setter 触发调用的时候，state array 就会把 state 变成 角标对应的那个位置的值

[![bFFSdx.png](https://s4.ax1x.com/2022/02/24/bFFSdx.png)](https://imgtu.com/i/bFFSdx)





#### 简单的实现

下面有一个 简单实现的例子

Note: 这不是 Hooks 的真正的实现方式，但是他能帮助你更好的理解Hooks, 这也是为什么会使用 模块级别的变量 等

``` jsx
let stateArr = []
let settersArr = []
let firstRun = true
let cursor = 0

export function useState(initVal) {
  if(firstRun) {
    stateArr.push(initVal)
    settersArr.push(createSetter(cursor))
    firstRun = false
  }
  
  const setter = settersArr[cursor]
  const state = stateArr[cursor]
  
  cursor++
  return [state, setter]
}

function createSetter(cursor) {
  return function setterWithCursor(newVal) {
    state[cursor] = newVal
  }
}

// our component code that uses hooks
function MyFunctionComponent() {
  const [firstName, setFirstName] = useState('Rudi') // cursor: 0
  const [lastName, setLastName] = useState('Yardley') // cursor: 1
  
  return (
    <div>
    	<Button onClick={() => setFirstName('Richard')}>Richard</Button>
      <Button onClick={() => setFirstName('Fred')}>Fred</Button>
    </div>
  )
}

function RenderMyComponent() {
  cursor = 0
  return <MyFunctionComponent />
}

console.log(state) // pre-render: []

RenderMyComponent()
console.log(state) // first render: ['Rudi', 'Yardley']

RenderMyComponent()
console.log(state) // Subsequent-render: ['Rudi', 'Yardley']

// click the 'Fred' button
console.log(state) // ['Fred', 'Yardley']


```





#### 为什么顺序很重要

如果我们 基于一些外部因素 甚至 组件状态 改变了render 周期 hooks 的顺序

让我们做一些 React团队 告诉你不要做的事情

```jsx
let firstRender = true

function RenderFunctionComponent() {
  let initName
  
  if(firstRender) {
    [initName] = useState('Rudi')
    firstRender = false
  }
  const [firstName, setFirstName] = useState(initName)
  const [lastName, setLastName] = useState('Yardley')
  
  return (
  	<Button onClick={() => setFirstName('Fred')}>Fred</Button>
  )
}
```

现在 我们把 `useState`的调用 放到了 条件判断里，让我们看一下他对程序造成的破坏

第一次渲染

[![bF3Mtg.png](https://s4.ax1x.com/2022/02/24/bF3Mtg.png)](https://imgtu.com/i/bF3Mtg)

目前为止， 我们的实例变量 `firstName`和`lastName`是正确的值，但是 让我们看看在第二次渲染时发生了什么

第二次渲染

[![bF3rcR.png](https://s4.ax1x.com/2022/02/24/bF3rcR.png)](https://imgtu.com/i/bF3rcR)

由于我们的状态存储不一致，现在`firstName` 和`lastName` 都变成了 `Rudi` , 这是很明显的错误，并且程序不能工作。但这也让我们明白了为什么 Hooks 的规则是这样的