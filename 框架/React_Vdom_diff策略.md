## React Vdom  及 diff 策略



**协调** : Virtual DOM 是一种编程概念。在这个概念里， UI 以一种理想化的，或者说“虚拟的”表现形式被保存于内存中，并通过如 ReactDOM 等类库使之与“真实的” DOM 同步。这一过程叫做协调



**协调过程 : **

![](.\1-1.png)

1. 初次渲染 根据 render（） 方法 返回的React 元素 ， 会创建一颗 React 元素树（VDOM）
2. 然后  根据 VDom  渲染出 真实的 DOM 
3. 当 组件 state 或 props 更新 时 ， 相同的 render 方法 会返回一颗 不同的树 
4. React 要 根据 这两颗 树 来 diff 出 差异（dom diff） 
5. 然后 将这些  差异更新到 真实的 DOM 上



**Virtual Dom** : 

Virtual DOM是对DOM的抽象，本质上是JavaScript对象，这个对象就是更加轻量级的对DOM的描述，提高重绘性能



**如何 把  JSX 转换 为 VDOM ： **

``` javascript
//已知  jsx 是 React.createElement 的语法糖 

// 例：

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

// 最终会被转换成如下结构

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



 

**diff 策略**

React 在以下两个假设的基础之上提出了一套 O(n) 的启发式算法: 

* 个不同类型的元素会产生出不同的树
* 开发者可以通过 `key` prop 来暗示哪些子元素在不同的渲染下能保持稳定

当对比两颗树时，React 首先比较两棵树的根节点。不同类型的根节点元素会有不同的形态

1. 对比不同类型的元素：

   当根节点 为 不同类型的元素时， React 会 卸载原有的树 并且 建立新的树，例如 当一个元素 从 <a> 变成 <img>  时 ， 会触发一个完整的重建流程

    当拆卸一棵树时，对应的 DOM 节点也会被销毁。组件实例将执行 `componentWillUnmount()` 方法。当建立一棵新的树时，对应的 DOM 节点会被创建以及插入到 DOM 中。组件实例将执行 `componentWillMount()` 方法，紧接着 `componentDidMount()` 方法。所有跟之前的树所关联的 state 也会被销毁

   ``` javascript
   // 如下情况 
   <div>
     <Counter />
   </div>
   
   <span>
     <Counter />
   </span>
   
   // Counter 组件也会先被卸载 ， 状态被销毁 ， 然后 重新装载一个 Counter组件
   ```

   

2. 对比相同类型元素

   对比两个相同类型的元素时， React 会保留 DOM 节点， 仅比对及更新有改变的属性
   
   ``` javascript
   <div className='before' title='title' />
   <div className='after' title='title' />
   // 通过对比这两个元素 ， react 知道只需要修改 dom 元素上的 classname 属性
       
   // 当 style 有差异时  也是如此 
   <div style={{color: 'red', fontWeight: 'bold'}} />
   <div style={{color: 'blue, fontWeight: 'bold'}} />
   // react 只会更改 color 样式  不会修改 fontWeight 
               
   // 在处理完当前节点之后，React 继续对子节点进行递归
   ```
   
   

3.  对比同类型的组件元素

   当一个组件更新时， 组件实例保持不变， 这样 在跨越不同的渲染时保持一致， React 将更新该组件实例的props 以跟最新的元素保持一致， 并且调用该实例的 componentwillreceiveprops 和 componentwillupdate 方法



   下一步 调用 render() 方法， diff 算法将在之前的结果以及新的结果中进行递归



 4. 对 子节点进行递归

    在默认情况下， 当递归 dom 节点的子元素时， react 会同时遍历两个子元素的列表， 当产生差异时， 生成一个 mutation 

    在子元素列表新增元素时， 变更开销比较小

    ```html
    <ul>
        <li>first</li>
        <li>senond</li>
    </ul>
    
    <ul>
        <li>first</li>
        <li>senond</li>
        <li>third</li>
    </ul>
    ```

    react 会先匹配两个 <li>first</li> 对应的树， 然后匹配 第二个 元素 <li>senond</li> 对应的树， 然后插入第三个元素的<li>third</li> 树

    如果简单实现的话， 在列表头部插入会很影响性能， 变更开销会比较大

    ``` html
    <ul>
        <li>first</li>
        <li>senond</li>
    </ul>
    
    <ul>
        <li>zero</li>
        <li>first</li>
        <li>senond</li>
    </ul>
    ```

    react 会针对每个子元素的mutate 而不是保持相同的 <li>first</li> 和 <li>senond</li> 子树完成， 这种情况下的低效可能导致性能问题



  5. keys

     为了解决以上问题 ， react 支持 key 属性， 当子元素拥有 key 时， react 使用 key 来匹配原有树上的子元素 以及最新树上的子元素，以下例子在新增 key 之后 使得之前的低效转换变得高效

     ``` html
     <ul>
     	<li key='2015'>Duke</li>
         <li key='2016'>Will</li>
     </ul>
     
     <ul>
         <li key='2014'>Conn</li>
     	<li key='2015'>Duke</li>
         <li key='2016'>Will</li>
     </ul>
     ```

     现在 react 只知道带着 2014 key 的元素 是新元素 ， 带着 2015 及 2016 的 key 元素 仅仅移动了



* 如何把 jsx 转换成 vdom
* 如何对比新旧vdom 
* 如何把 vdom 的 change 更新到 real dom 上去 

