## React Fiber

Fiber 是 从 React 16 开始  的一种新的架构方式, 其目的是为了优化  react 项目的性能问题



在 React 16 之前的版本 组件中 当 使用 setState（） 更新组件状态时，React 会以当前 调用 setState() 方法的 组件作为根节点 去更新整个子树，我们能做的只有在某个子组件中使用 shouldComponentUpdate 来阻止组件的重新渲染， 在这个 自顶向下的 递归（diff, 更新， 渲染）的过程中， 是无法中断的，js 会一直占用主线程， 这就可能导致 主线程上 和 UI 相关的 布局， 动画 等任务 无法及时处理，  导致  掉帧， 卡顿等问题



#### React元素 与 Fiber 节点

从组建中 Render 方法 返回的 jsx 就是 React 元素的集合 ， jsx 是 描述 React 元素（React.createElement）的语法糖

``` javascript
// jsx
render() {
    return(
    	<span key='1'> react </span>
    )
}

// 等价于 React.createElement
render() {
    return [
        React.createElement(
        	'span',
            {
                key: '1'
            },
            'react'
        )
    ]
}

// React.createElement 会被编译成如下结构

[
    {
        $$typeof: Symbol(react.element),
        type: 'span',
        key: '1',
        props: {
            children: 0
        }
    }
]
```

​    

---------------------------------------

占位 图 1-1

-----------------------------------------------------------





在 组件 更新 渲染期间 从 render 方法里返回的每个React 元素都会有一个相应的 fiber 节点 ， 形成一颗 fiber 树， 这样 整个 更新 渲染的过程 就会因为 fiber 节点  而变为 可拆分的一个个小任务 之后 每次只做一小段，做完一段就把时间控制权交还给主线程，而不像之前长时间占用。这种策略叫做cooperative scheduling（合作式调度）

每一个 fiber 节点 都拥有 三个属性 

* return :  父节点
* child : 第一个子组件
* sibling : 右边的兄弟组件

这样 一颗 fiber tree  就变成了一个 链表， 以实现 深度优化遍历



