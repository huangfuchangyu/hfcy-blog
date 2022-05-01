## React 运行时环境

话不多说，让我们深入开始理解React 吧



#### 宿主树

React 程序通常会输出一颗会随时间变化的树。它有可能是一颗DOM树，IOS视图层，PDF原语，又或是JSON对象。然而，我们通常会用它来展示UI，我们称它为 “宿主树” ，因为它往往是React 之外宿主环境中的一部分，就像DOM或IOS。宿主树通常由他自己的命令式API。而React 就是它上面的那一层。

React 坚持两个原则： 

1. 稳定性： 宿主树是相对稳定的，大多数情况的更新并不会从根本上改变其整体结构。
2. 通用性： 宿主树可以被拆分为外观和行为一致的UI模式而不是随机的形状

这些原则恰好适用于大多数UI，然而，当输出没有稳定的模式时，React 并不适用。例如 React也许适合帮助编写Twitter 客户端， 但对于一个 “3D管道屏幕保护程序” 并不会起太大作用

#### 宿主实例

宿主树由节点组成，我们称之为“宿主实例”

在DOM环境中，宿主实例就是我们通常所说的DOM节点，就像你调用 `document.createElement('div')` 时获得的对象。在IOS中，宿主实例可以是从JavaScript到原生视图唯一标识的值。

宿主实例有他们自己的属性（例如 domNode.className 或者 view.tintColor ）他们也有可能将其他的宿主实例作为子项

通常会有原生的API 用于操作这些宿主实例，例如 在DOM 环境中会提供像 `appendChild` `removeChild` `setAttribute` 等一系列的API , 在 React 应用中， 通常你不会调用这些api ，因为那是 React 的工作。



#### 渲染器

渲染器教会React 如何与特定的宿主环境通信以及如何管理它的宿主实例。React DOM , React Native 都可以称作 React 渲染器。也可以创建自寄的 React 渲染器。

React 渲染器 能以下面两种模式之一进行工作。

绝大多数渲染器都被用作 “突变” 模式， 这种模式正是 DOM 的工作方式： 我们可以创建一个节点，设置它的属性，在之后往里面增加或者删除子节点，宿主实例是完全可变的

但 React 也能以 “不变” 模式工作, 这种模式适用于那些并不提供像 `appendChild` 的API 而是可伶双亲树 并始终替换掉顶级子树的宿主环境，在宿主树级别上的不可变性使得多线程变得更加容易, `React Fabric` 就利用了这一模式。





#### React 元素

在宿主环境中， 一个宿主实例（如 DOM 节点）是最小的构建单元， 而在 React 中，最小的构建单元是 React 元素

React 元素是一个普通的 JavaScript 对象， 他用来描述一个宿主实例

``` jsx
// jsx 是用来描述这些对象的语法糖
// <button className="blue" />

{
  type: 'button',
  porps: {class: 'blue'}
}
```

React 元素是轻量级的因为没有宿主实例与他绑定在一起，同样的，他只是对你想要在屏幕上看到的内容的描述

就像宿主实例一样，React 元素也能形成一棵树

``` jsx
// jsx 是用来描述这些对象的语法糖
// <dialog>
// 	<button class="blue" />
//   <button class="red" />
// </dialog>
{
  type: 'dialog',
  props: {
    children: [
      {
        type: 'button',
        props: {className: 'blue'}
      },
      {
        type: 'button',
        props: {className: 'red'}
      }
    ]
  }
}
```

但是 请记住 React 元素并不是永远存在的, 它总是在重建和删除之间不断循环着

React 元素具有不可变性，例如，你不能改变React 元素中的子元素或者属性，如果你想要在稍后渲染一些不同的东西，你需要从头创建新的React 元素并描述它



#### 入口

每一个 React渲染器 都有一个 入口， 正是那个特定的API让我们告诉React ，将特定的React 元素树渲染到真正的宿主实例中去。

例如，React DOM 的入口就是 `ReactDOM.render`：

``` jsx
ReactDOM.render(
	// {type: 'button', props: {className: 'blue'}}
  <button className="blue" />,
  document.getElementById('container')
)
```

React 会查看 `reactElement.type` （在例子中是 button） 然后告诉React DOM 渲染器创建对应的宿主实例并设置正确的属性：

``` jsx
// 在 ReactDOM 渲染器内部（简化版）
function createHostInstance(reactElement) {
  let domNode = document.createElement(reactElement.type)
  domNode.className = reactElement.props.className
  return domNode
}
```

在我们的例子中，React 会这样做

``` jsx
let domNode = document.createElement('button')
domNode.className = 'blue'
domContainer.appendChild(domNode)
```

如果React 元素在 `createElement.props.children` 中含有子元素， React 会在第一次渲染中递归地为他们创建宿主实例







#### 协调

如果我们用同一个 container 调用 `ReactDOM.render()` 两次 会发生什么

``` jsx
ReactDOM.render(
	<button className="blue" />,
  document.getElementById('container')
)

// 应该替换掉 button 宿主的实例吗
// 还是在已有的 button 上更新属性
ReactDOM.render(
	<button className="red" />,
  document.getElementById('container')
)
```

同样的 React 的工作是将 React 元素树映射到宿主树上去， 确定该对宿主实例做什么来响应新的信息，叫做 **协调（reconciler）**

有两种方法可以解决它，简化版的React 会丢弃已经存在的树然后从头开始创建它

``` jsx
let domContainer = document.getElementById('container')
// 清除掉原来的树
docContainer.innerHTML = ''
// 创建新的宿主实例
let domNode = document.createElement('button')
domNode.className = 'red'
domContainer.appendChild(domNode)
```

但是在DOM 环境下，这样的做法效率低下而且会丢失像 focus， selection， scroll 等许多状态，相反，我们希望React 这样做：

``` jsx
let domNode = domContainer.firstChild
// 更新已有的宿主实例
domNode.className = 'red'
```

换句话说，React 需要决定何时更新一个已有的宿主实例来匹配新的React元素，何时该重新创建新的宿主实例

这就引出了一个识别问题，React 元素可能每次都不同，到底什么时候该从概念上引用同一个宿主实例呢

在我们的例子中，它很简单， 我们之前渲染了`<button>` 作为第一个的子元素，接下来我们想要在同一个地方再次渲染`<button>` 在宿主实例中我们已经有了一个 `<button>` 为什么还要重新创建呢，让我们重用它

这与React 如何思考并解决这类问题已经很接近了

**如果相同的元素类型在同一个地方先后出现2次，React 会重用已有的宿主实例**

这里有一个例子 ，大致解释了React 是如何工作的

``` jsx
// let domNode = document.createElement('button')
// domNode.className =  'blue'
// domContainer.appendChild(domNode)
ReactDOM.render(
	<button className="blue" />,
  document.getElementById('container')
)
// 能重用宿主实例吗 ？  能 （button -> button）
// domNode.className = 'red'
ReactDOM.render(
	<button className="red" />,
  document.getElementById('container')
)

// 能重用宿主实例吗 不能 （button -> p）
// domContainer.removeChild(domNode)
// domNode = document.getElement('p')
// domNode.textContent = 'Hello'
// domContainer.appendChild(domNode)
ReactDOM.render(
  <p>hello</p>,
  document.getElementById('container')
)

// 能重用宿主实例吗 能 （p -> p）
// domNode.textContent = 'Bye'
ReactDOM.render(
  <p>Bye~</p>,
  document.getElementById('container')
)
```

同样的启发方式也适用于子树，例如 ，当我们在 `<dialog>` 中新增两个 `<button>` , React 会先决定是否要重用 `<dialog>` ，然后为每一个子元素重复这个决定步骤





#### 条件

如果React 在渲染更新前后只重用那些元素类型匹配的宿主实例，当遇到包含条件语句的内容时又该重新渲染呢

假设我们只想首先展示一个输入框，但之后要在它之前渲染一条信息

``` jsx
// 第一次渲染
ReactDOM.render(
  <dialog>
  	<input />
  </dialog>,
  domContaner
)

// 第二次渲染
ReactDOM.render(
  <dialog>
  	<p>i was just here</p>
    <input />
  </dialog>,
  domContainer
)
```

在这个例子中， `<input>` 宿主实例会被重新创建, React 会遍历整个元素树， 并将其与先前的版本进行比较：

* `dialog -> dialog` ： 能重用宿主实例吗？ 能 - 因为类型是匹配的
* `input -> p`： 能重用宿主实例吗  不能，类型改变了 需要删除已有的 `input` 然后重新创建一个 `p` 宿主实例
* `(nothing) -> input `： 需要重新创建一个 `input` 宿主实例

因此， React 会像这样执行更新:

``` jsx
let oldInputNode = dialogNode.firstChild
dialogNode.removeChild(oldInputNode)

let pNode = document.createElement('p')
pNode.textContent = 'i was just here'
dialogNode.appendChild(pNode)

let newInputNode = document.createElement('input')
dialogNode.appendChild(newInputNode)
```

这样的做法并不科学，因为事实上， `<input>` 并没有被 `<p>` 所替代，他只是移动了位置而已， 我们不希望因为重建DOM 而丢失了 selection， focus 等状态

虽然这个问题容易解决，但是这个问题在React 中并不常见

事实上，你很少会直接调用`ReactDOM.render` 相反， 在React 程序中往往会被拆分成这样的函数

```jsx
function Form({showMessage}) {
  let message = null
  if(showMessage) {
    message = <p>i was just here</p>
  }
  return (
    <dialog>
    	{message}
      <input />
    </dialog>
  )
}
```

这个例子 并不会遇到刚刚我们描述的问题i， 让我们用对象注释而不是 JSX 也许可以更好第理解其中的原因， 来看一下 `dialog` 中的 子元素树

``` jsx
function Form ({showMessage}) {
  	let message = null
    if(showMessage) {
      message = {
        type: 'p',
        props: {children: 'i was just here'}
      }
    }
  
  	return {
      type: 'dialog',
      props: {
        children: [
          massage,
          {type: 'input', props: {}}
        ]
      }
    }
}
```

不管showMessage 是 true 和 false ，在渲染过程中 `input` 总是在children 的第二个位置，并且不会改变

如果 `showMessage` 从 false 改为 true ，React 会遍历整个元素树， 并与之前的版本进行比较

* `dialog -> dialog` ： 能够重用宿主实例吗， 能  因为类型匹配
* `(null) -> p`： 需要插入一个新的 `p` 宿主实例
* `input -> input` ： 能够重用宿主实例吗  能 因为类型匹配

这样一来 输入框的状态就不会丢失了







#### 列表

比较树中同一位置元素类型对于是否该重用还是重建相应的宿主实例往往已经足够

但是只适用于当子元素是静止的并且不会重排序的情况，在上面的例子中，即使`message` 不存在，我们仍然知道输入框在消息之后，并且再没有其他的子元素，而当遇到动态列表时， 我们不能确定其中的顺序总是一成不变的

``` jsx
function ShoppingList({list}) {
  return (
    <form>
    	{
        list.map(item => {
          <p>
          	you bougnt {item.name}
            <br />
            enter how many you want<input />
          </p>
        })
      }
    </form>
  )
}
```

如果我们的商品列表被重新排序了， React 只会看到所有的 `p` 以及里面的 `input` 拥有相同的类型， 并不知道改如何移动他们。（在React 看来，虽然这些商品本身改变了，但是他们的顺序并没有改变）

所以 React 会对这是个商品进行类似如下的重排序

``` jsx
for(let i=0; i<10; i++) {
  let pNode = formNode.childNodes[i]
  let textNode = pNode.firstChild
  textNode.textContent = 'you bought' + items[i].name
}
```

React 只会对其中的每个元素进行更新而不是将其重新排序，这样做会造成性能上的问题和潜在的bug ， 例如当商品列表的顺序改变时， 原本在第一个输入框的内容仍然会存在于第一个输入框中，尽管事实上在商品列表里他应该代表着其他商品

这就是为什么每次当输出中包含元素数组是，React 都会让你指定一个叫做`key` 的属性

``` jsx
function ShoppingList({list}) {
  return (
    <form>
    	{
        list.map(item => {
          <p key={item.productId}>
            you bougnt {item.name}
              <br />
            enter how many you want<input />
          </p>
        })
      }
    </form>
  )
}
```

`key` 给予React 判断子元素是否真正相同的能力，即使在渲染前后他在父元素中的位置不是相同的

当React 在 `<form>` 中发现 `<p key="42">`, 它就会检查之前版本中的 `<form>` 是否同样含有`<p key="42">` 。即使`<form>` 中的子元素们改变位置后，这个方法同样有效，在渲染前后 key 仍然相同时，React 会重用先前的元素实例 ，然后重新排序其兄弟元素

需要注意的是`key` 只与特定的父亲React 元素相关联，比如 `form` ，React 并不会去匹配父元素不同但key 相同的子元素





#### 纯净

React 组件中对于 props 应该是纯净的

``` jsx
function Button(props) {
  // 没有作用
  props.isActive = true
}
```

通常来说，突变在React中不是惯用的

不过 局部的突变是被允许的

```jsx
function FriendList({firends}) {
  let items = []
  for(let i=0; i<firends.length; i++) {
    let firend = firends[i]
    items.push(
    	<Friend key={friend.id} friend={friend}>
    )
  }
  return <section>{items}</section>
}
```

当我们在函数组件内部创建 `items` 时不管怎样改变它都行，只要这些突变发生在将其作为最后的渲染结果之前。所以并不需要重写你的代码来避免局部突变

同样地，惰性初始化是被允许的即使它不是完全“纯净” 的

``` jsx
function ExpenseForm() {
  // 只要不影响到其他组件这是被允许的
  SuperCalculator.initializeIfNotReady()
  // ...继续渲染
}
```

只要调用组件多次是安全的，并且不会影响其他组件的渲染，React 并不关心你的代码是否像严格的函数式变成一样百分百纯净，在React 中，幂等性比纯净性更加重要

也就是说，在React 组件中不允许有用户可以直接看到的副作用，换句话说，仅调用函数式组件时不应该在屏幕上产生任何变化





#### 递归

我们该如何在组件中使用组件，组件属于函数因此我们可以直接进行调用

```jsx
let reactElement = Form({showMessage: true})
ReactDOM.render(reactElement, domContainer)
```

然而，在React 运行时中这并不是惯用的思维方式

相反，使用组件惯用的方式与我们已经了解的机制相同，即React 元素, 这意味着不需要你直接调用组件函数，React 会在之后为你做这件事情

``` jsx
// {type: Form, props: {showMessage: true}}
let reactElement = <Form showMessage={true} />
ReactDOM.render(reactElement, domContainer)
```

然后在React内部，你的组件会被这样调用

```jsx
let type = reactElement.type // Form
let props = reactElement.props // {showMessage: true}
let result = type(props) // 无论Form 会返回什么
```

组件函数名称按照规定需要大写，当JSX 转换时看见`<From>`而不是`<form>` 他让对象`type` 本身称为标识符而不是字符串

``` jsx
console.log(<form />.type) // 'form' 字符串
console.log(<Form />.type) // Form 函数
```

我们并没有全局的注册机制， 字面上当我们输入`<Form>` 时代表着 `Form`。如果`Form` 在局部作用域中并不存在, 你会发现一个 javascript 错误，就像平常你使用错误的变量名称一样

因此，当元素类型是一个函数的时候，React 会做什么呢，他会调用你的组件，然后询问组件想要渲染什么元素

这个步骤会递归式地执行下去，总的来说 ， 他会这样执行： 

* 你： `ReactDOM.render(<APP />, domContainer)`



* React: APP  你想要渲染什么

  * APP： 我要渲染 包含 `<Content>` 的 `<Layout>`

  

* React: `<Layout>` 你要渲染什么

  * Layout: 我要在 div 中渲染我的族元素， 我的子元素是 `<Content>` 所以他应该渲染到 div 中去

  

* React: `<Content>` 你要渲染什么

  * Content : 我要在 `<article>` 中渲染一些文本和 `<Footer>`

  

* React: `<Footer>` 你要渲染什么

  * Footer: 我要渲染含有文本的 `footer`

  

* React ： 好的 让我们开始吧

``` jsx
// 最终的DOM结构
<div>
	<article>
  	Some text
    <footer>some more text</footer>
  </article>
</div>
```

这就是为什么我们说协调是递归式的，当React 便利整个元素树时， 可能会遇到元素的 `type` 是一个组件，React 会调用它然后继续沿着返回的React 元素下行， 最终我们会调用完所有的组件，然后 React 就会知道改如何改变宿主树

如果在同一位置`type` 改变了 （由索引和可选 `key` 绝决定），React 会删除其中的宿主实例并将其重建





#### 控制反转

你也许会好奇，为什么我们不直接调用组件，为什么要编写`<Form />` 而不是 `Form()` 

React 能够做的更好如果他 “知晓”  你的组件而不是在你递归调用他们之后生成的React 元素树

``` jsx
// React 并不知道 Layout 和 Article 的存在
// 因为你在调用他们
ReactDOM.render(
	Layout({children: Article()}),
  domContainer
)

// React 知道Layout 和 Article 的存在
// React 来调用他们
ReactDOM.render(
  <Layout><Article /></Layout>,
  domContainer
)
```

这还是一个关于 控制反转 的经典案例，通过让 React 调用我们的组件， 我们会获得一些有趣的属性：

* **组件不仅仅只是函数。**React 能够用在树中与组件本身紧密相连的局部状态等特性来增强组件功能，优秀的运行时提供了与当前问题相匹配的基本抽象，就像我们已经提到过的，React 专门针对那些渲染UI树并且能够响应交互的应用，如果你直接调用了组件，你就只能自己来构建这些特性了
* **组件类型参与协调。**通过React 来调用你的组件， 能让他了解更多关于元素树的结构，例如，当你从渲染 `<Feed>` 页面转到 `Profile` 页面， React 不会尝试重用其中的宿主实例， 就像你用 `<p>` 替换掉 `<button>` 一样。所有的状态都会丢失，对于渲染完全不同的视图时，通常来说这是一件好使，你不会想要在`<PassowrdForm>` 和 `<MessagerChat>` 之间保留输入框的状态， 尽管 `<input>` 的位置意外地“排列”在他们之间
* **React 能够推迟协调。**如果让React控制调用你的组件， 他能做很多有趣的事情，例如，他可以让浏览器在组件调用之间做一些工作，这样重渲染大体量的组件树时就不会阻塞主线程， 想要手动编排这个过程而不依赖React 的话将会十分困难
* **更好的可调试性。**如果组件是库中所重视的一等公民，我们就可以构建丰富的开发者工具，用于开发中自省

让React 调用你的组件函数还有最后一个好处就是惰性求值，让我们看看他是什么意思





#### 惰性求值

当我们在 javascript 中调用函数时， 参数往往在函数调用之前被执行

``` javascript
// 他会后执行
eat(
  // 首先他会先执行
	preparemeal()
)
```

这通常是JavaScript 开发者所期望的因为 JavaScript函数可能有隐含的副作用，如果我们调用了一个函数，但直到它的结果不知怎的被使用后该函数扔没有执行，这会让我们感到十分诧异

但是 React 组件是相对纯净的，如果我们知道它的结果不会在屏幕上出现， 则完全没有必要执行它

看下面这个含有 `<Comments>` 的 `<Page>` 组件:

``` jsx
function Story({currentUser}) {
  return (
  	<Page user={currentUser}>
    	<Comments />
    </Page>
  )
}
```

`<Page>` 组件能够在 `<Layout>` 中渲染传递给他的子项

``` jsx
function Page({currentUser, children}) {
  return (
  	<Layout>
    	{children}
    </Layout>
  )
}
```

> 在 JSX 中 `<A><B /></A>` 和 `<A children={<B />} />` 相同

但是要是存在提前返回的情况呢

``` jsx
function Page({currentUser, children}) {
  if(!currentUser.isLogin) {
    return <h1>please login</h1>
  }
  
  return (
  	<Layout>
    	{children}
    </Layout>
  )
}
```

让 React 来决定何时以及是否调用组件，如果我们的`Page` 组件忽略自身的 children prop 且相反地渲染了 `<h1>please login</h1>` , React 不会尝试去调用 `Comments` 函数 



#### 一致性

即使我们想将协调过程本身分割成非阻塞的工作块，我们仍然需要在同步的循环中对其真实的宿主实例进行操作，这样我们才能保证用户不会看见半更新状态的UI，浏览器也不会对用户不应看到的中间状态进行不必要的布局和样式的重新计算

这也是为什么React 将所有的工作分成了 “渲染阶段” 和 "提交阶段" 的原因， 渲染阶段是当 React 调用你的组件然后进行协调的阶段，在此阶段进行干涉是安全的且在 未来这个阶段将会变成异步的，提交阶段就是 React 操作宿主树的时候，这个阶段永远是同步的。





#### 缓存

当父组件通过 `setState` 准备更新时，React 默认会协调整个子树，因为React 并不知道在父组件中是否会影响到子代，所以React 默认保持一致性，这听起来会有很大的性能消耗，但事实上，对于小型和中型的子树来说，这并不是问题

当树的深度和广度达到一定程度时，你可以让React 去缓存子树，并且重用先前的渲染结果，当prop 在浅比较之后是相同时：

``` jsx
function Row({item}) {
  // ...
}
export default React.memo(Row)
```

现在 父组件`<Table>` 中调用 `setState` 时如果 `<Row>` 中的  `item`与先前渲染的结果是相同的 ，React 就会直接跳过协调的过程

默认情况下，React 不会故意缓存组件，许多组件在更新的过程中总是会接收到不同的props，所以对他们进行缓存只会造成净亏损





#### 原始模型

令人讽刺的是，React 并没有使用 “反应式” 系统来支持细粒度的更新，换句话说，任何在顶层的更新只会触发协调而不是局部更新那些受影响的组件

这样的设计是有意为之的，对于web 应用程序来说，“交互时间” 是一个关键指标，而通过遍历整个模型去设置细粒度的监听器只会浪费宝贵的时间，此外在很多应用中交互往往或导致或小或大的更新，因此细粒度的订阅只会浪费内存资源

有那么一些应用细粒度的订阅对他们来说是有用的，例如股票代码，这是一个极少见的例子，因为所有的东西都需要在同一时间内持续更新，虽然命令式的方法能够优化此代码，但 React 并不适用于这种情况，同样的 ，如果你想要解决该问题，你就得在React 之上实现细粒度的订阅

注意 ，即使细粒度的订阅和反应式的系统也无法解决一些常见的性能问题，例如 渲染一颗很深的树，而不阻塞浏览器，改变跟踪并不会让他变得更快，这样只会变得更慢因为要进行一些额外的订阅工作，另一个问题是我们需要等待返回的数据在渲染视图之前，在React 中我们用 “并发渲染” 来解决这些问题





#### 批量更新

一些组件也许想要更新状态来响应同一事件，下面这个例子是假设的，但是却说明一个常见的模式

``` jsx
function Parent() {
  let [count, setCount] = useState(0)
  
  return(
  	<div onClick={() => setCount(count + 1)}>
    	Praent clicked {count} times
      <Child></Child>
    </div>
  )
}

function Child() {
  let [count, setCount] = useState(0)
  
  return(
  	<div  onClick={() => setCount(count + 1)}>
    	Child clicked {count} times
    </div>
  )
}
```

当事件被触发时，子组件的`onClick` 首选被触发，然后父组件在他自己的 `onClick` 中调用 `setState`

如果React 立即重新渲染组件以响应`setState` 调用，最终我们会重新渲染子组件两次

``` jsx
// 进入 React 浏览器 click 事件处理过程
Child (onClick)
	- setState
	-re-render Child   // 不必要的渲染
Parent(onClick)
	-setState
	-re-render Praent
  -re-render Child
  
// 结束 React 浏览器的 click 事件处理
```



第一次 Child 组件渲染时浪费的，并且我们也不会让React 跳过 Child 的第二次渲染，因为 Parent 可能会传递不同的数据 ，由于其自身状态的更新

这就是为什么React 会在组件内所有事件触发完成后进行批量更新的原因



``` jsx
// 进入 React 浏览器 click 事件处理过程
Child (onClick)
	- setState
Parent(onClick)
	-setState

// Porcessing state updates
	-re-render Praent
  -re-render Child
  
// 结束 React 浏览器的 click 事件处理
```



组件内调用 setState 并不会立即进行重新渲染，相反 React 会先触发所有的事件处理器，然后再触发一次重渲染已进行所谓的批量更新

批量更新虽然有用，但可能会让你感到惊讶，如果你的代码这样写

``` jsx
const [count, setCount] = useState(0)

function increment() {
  setCounter(count + 1)
}

function handleClick() {
  increment()
  increment()
  increment()
}
```

如果我们将 count 初始值设为 0 ，上面的代码只会代表三次 setCount(1) 调用， 为了解决这个问题 我们给 setState 提供了 一个  “update” 函数作为参数

``` jsx
const [count, setCount] = useState(0)

function increment() {
  setCounter(c => c + 1)
}

function handleClick() {
  increment()
  increment()
  increment()
}
```



React 会将 updater 函数放入队列中，并在之后按顺序执行他们， 最终 count 会被设置成3 并作为一次重渲染的结果

当逻辑状态变得复杂，而不仅仅只是少数的 setState 调用时， 我建议你用 `useReducer Hook` 来描述你的局部状态， 他就像 updater 的升级模式在这里你能给每一次更新命名

``` jsx
const [counter, setCounter] = useReducer((state, action) => {
  if(actioin === 'increment') {
    return state + 1
  }
})

function handleClick() {
  dispatch('increment')
  dispatch('increment')
  dispatch('increment')
}
```







#### 调用树

编程语言的运行时往往有 调用栈， 当 函数 `a()` 调用 `b()` `b()` 又调用 `c()` ，在 JavaScript 引擎中 会有像 `[a,b,c]` 这样的数据结构来跟踪当前的位置以及接下来要执行的代码， 一但 `c` 函数执行完毕， 它的调用栈就消失了，因为不在需要了  ， 然后返回到 `b`函数中，当我们结束函数 `a` 的执行时，调用栈就会被清空

当然，React 以 JavaScript 运行当然也遵循 JavaScript 的规则， 但是我们可以想象在 React 内部有自己的调用栈来记忆我们当前正在渲染的组件，例如`[App, Page, Article]`

React 与通常意义上的编程语言进行时不同，因为它针对于渲染 UI 树，这些树需要保持“活性”，这样才能使我们与其进行交互，在第一次 `ReactDOM.render()` 出现之前，DOM操作并不会执行

当我们调用完 `Article` 组件，它的 React 调用树 并没有被摧毁，我们要将局部状态保存以便映射到宿主实例的某个地方

这些调用树帧会随他们的局部状态和宿主实例一起被摧毁，但是只会在 协调规则认为这是必要的时候执行，如果你曾读过React 源码，这些帧实际上就是 `Fibers`

Fibers 是局部状态真正存在的地方， 当状态被更新后， React 将其下面的Fibers 标记为需要进行协调，之后便会调用这些组件







#### 上下文

在React 中，我们将数据作为props 传递给其他组件，有些时候，大多数组件需要相同的东西， 例如 当前选中的可视主题，将它一层层专递会非常麻烦

我们通过 `Context` 来解决这个问题，他就像组件的动态范围，能让你从顶层传递数据，并让每个子组件在底部能够读取到值，当值变化时，还能重新渲染

``` jsx
const ThemeContext = React.createContext(
'light'
)

function DarkApp() {
  return(
    <ThemeContext.Provider value="dark">
    	<MyComponents />
    </ThemeContext.Provider>
  ) 
}

function SomeDeepChild() {
  const theme = useContext(ThemeContext)
}

// 当 SomeDeepChild 渲染时， useContext 会寻找树中最近的 ThemeContext.Provider 并使用它的value
// 如果没有 ThemeContext.Provider ， useContext 调用结果就会被  createContext 时传递的默认值所取代
```







#### 自定义 HOOKS 

由于`useState` 和 `useEffect` 是函数调用，因此我们可以将其组合成自寄的 Hooks

``` jsx
function MyResponsieComponent() {
  // 自定义Hook
  const width = UseWindowWidth()
  return(
  	<p>window width is {width}</p>
  )
}

function UseWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth)
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handleResize)
    return () => {
      window.removeEventListener('resize', handleResize)
    }
  })
}
```

自定义Hooks 让不同的组件共享可重用的状态逻辑，注意状态本身是不共享的，每次调用Hook 都只声明了其自身的独立状态







#### 静态使用顺序

你可以把 `useState` 想象成一个可以定义 React 状态变量 的语法， 他并不是真正的语法， 当然我们仍在用 JavaScript 编写应用，但是我们将 React 作为一个运行时环境来看待, 因为 React 用 JavaScript 来描绘整个UI 树，它的特性往往更接近于 语言层面

假设 `use ` 是语法

``` jsx
component Example(props) {
  // 这不是真正的语法
  const [count, setCount] = use State(0)
  return(
  	// ...
  )
}
```

当然 `use` 并不是真正的语法

然而 React 的确期望所有的Hooks 调用只会发生在 组件的顶部并且不在条件语句中，这些 Hooks 的规则 能够被 `linter plugin` 所规范

Hooks 的内部实现其实是链表，当你调用 `useState` 的时候 ， 我们将指针移动到下一项，当我们退出组件的 "调用树"帧时， 会缓存该结果的列表直到下次渲染开始

数组 也许是比链表更好解释其原理的模型

``` javascript
// 伪代码
let hooks, i
function useState() {
  i++
  if(hooks[i]) {
    再次渲染
    return hooks[i]
  }
  // 第一次渲染
  hooks.push(...)
}
             
// 准备渲染
i = -1
 hooks = fiber.hooks || []
  // 调用组件
  YourComponent()
  // 缓存 Hooks 的状态
  fiber.hooks = hooks
```





