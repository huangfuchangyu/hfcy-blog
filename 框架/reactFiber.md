## React Fiber

Fiber 是 从 React 16 开始  的一种新的架构方式, 其目的是为了优化  react 项目的性能问题



在 React 16 之前的版本 组件中 当 使用 setState（） 更新组件状态时，React 会以当前 调用 setState() 方法的 组件作为根节点 去更新整个子树，我们能做的只有在某个子组件中使用 shouldComponentUpdate 来阻止组件的重新渲染， 在这个 自顶向下的 递归（diff, 更新， 渲染）的过程中， 是无法中断的，js 会一直占用主线程， 这就可能导致 主线程上 和 UI 相关的 布局， 动画 等任务 无法及时处理，  导致  掉帧， 卡顿等问题



#### React元素 与 Fiber 节点

从组建中 Render 方法 返回的 jsx 就是 React 元素的集合 ， jsx 是 描述 React 元素（React.createElement）的语法糖



在 组件 更新 渲染期间 从 render 方法里返回的每个React 元素都会有一个相应的 fiber 节点 ， 形成一颗 fiber 树， 这样 整个 更新 渲染的过程 就会因为 fiber 节点  而变为 可拆分的一个个小任务 之后 每次只做一小段，做完一段就把时间控制权交还给主线程，而不像之前长时间占用。这种策略叫做cooperative scheduling（合作式调度）

每一个 fiber 节点 都拥有 三个属性 

* return :  父节点
* child : 第一个子组件
* sibling : 右边的兄弟组件

这样 一颗 fiber tree  就变成了一个 链表， 以实现 深度优化遍历

![](.\100.png)

转换成 Fiber 结构

![](.\200.png)

Fiber 是一种数据结构(堆栈帧)，也可以说是一种解决可中断的调用任务的一种解决方案，它的特性就是**时间分片(time slicing)和暂停(supense)**



**如何利用浏览器空闲时间？**

requestIdleCallback， 方法将在浏览器的空闲时段内调用的函数排队。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件

``` javascript
var handle = window.requestIdleCallback(callback[, options])
// 返回值 : 一个ID，可以把它传入 Window.cancelIdleCallback() 方法来结束回调
// callback: 一个在事件循环空闲时即将被调用的函数的引用
// options: timeout,如果指定了timeout并具有一个正值，并且尚未通过超时毫秒数调用回调，那么回调会在下一次空闲时期被强制执行，尽管这样很可能会对性能造成负面影响

// eg: 
var handle = window.requestIdleCallback(
	_ => {
        // dosomething
    },
    {timeout: 1000}
)
```



**Fiber 是如何工作的？**