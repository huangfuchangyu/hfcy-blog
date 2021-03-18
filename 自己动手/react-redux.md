## react-redux



`react-redux` 是在 `redux` 的基础上 ，  结合 `react context` 



只需要 关注 两个 `API `

* `<Provider store>`
* `connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`





#### `<Provider store>`

用法： 

``` javascript
import { Provider } from 'react-redux'

render() {
    return (
      <Provider store={Store}>
        <AppContainer />
      </Provider>
    )
  }
```

其实 他就是 包了一层 `react`的  ` Context.Provider`



关于 `react context ` 的用法如下

``` jsx

// 创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 Provider 中读取到当前的 context 值
const MyContext = React.createContext(defaultValue)

// 每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化
<MyContext.Provider value={某个值 }>
    
<MyContext.Consumer>
  {value => 基于 context 值进行渲染}
</MyContext.Consumer>
```



`react-redux` 的 `provider` 应该是如下实现

``` jsx
return() {
    <ReactContext.Provider value={store}>
    	{props.children}
    </ReactContext.Provider>
}
```

其实就是 将 `redux ` 的 `store`  放到 `react context` 上 然后 传递下去



#### `connect()`

重点看 `connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`

连接 `React `组件与` Redux store`

连接操作不会改变原来的组件类
返回 一个新的已与 `Redux store `连接的组件类

先看 用法

``` jsx
export default connect(
  reduxState => {
    return {
      ...reduxState,
    }
  },
  {
    reduxAction
  },
)(Home)
```

根据调用方式 ， 第一版实现

``` jsx
function connect(mapStateToProps, mapDispatchToProps) {
    
    return function fnHoc(comp) {
        // 两次调用， 然后 返回一个不改变原来组件类的组件类
        return function fnConnect() {
            // 把 传入的组件的props 拿出来 ， 再 结合 redux state  和 dispatch 传回去
            const compProps = comp.props
            
            // 获取context的值
      		const context = useContext(ReactReduxContext)
			// 解构出store
      		const { store } = context
            // 拿到state
      		const state = store.getState()  
            
            // mapStateToProps 是一个方法  ， 返回 state
            const stateProps = mapStateToProps(state)
            
            const finalProps = Object.assign({}, stateProps, mapDispatchToProps,compProps)
            
             return <WrappedComponent {...finalProps} />
        }
    }
}
```



但是 这里 还有 两个问题： 

1. 怎么判断状态变化 并且怎么更新组件
2. 组件更新的顺序问题



#### 怎么判断状态变化

通过 `redux` 中的 `subscribe` 和 `getState`,  结合 上一次 组件 的 `state`  对比



``` jsx
// 记录上次渲染参数
const lastChildProps = useRef()
useLayoutEffect(() => {
  lastChildProps.current = actualChildProps
}, [])
```

``` jsx
store.subscribe(() => {
    // redux 状态更新了
    const stateCpy = store.getState()
})
```

这样 新旧state 就可以获取到了 

然后 利用 `this.setState({})` 或者 `forceUpdate()`  函数组件 利用`useReducer` 强制更新





#### 组件更新顺序问题

由上面的实现， 我们可知， `connect` 把 组件 和 `redux` 连接了起来， 只要 `redux` 的状态 与 本地状态不一致（浅比较）， 就会触发组件强制更新， 但是 这有违 `react` 数据流，  自上而下的数据流

`react-redux` 的解决方式是 通过 一套`pub sub` `Subscription` 

`connect` 方法 会判断 自己的 `props` 上是否有 `subcription` 如果没有， 则认为是根组件， 则订阅 `store.subscribe()`

如果有， 则认为 不是根组件， 订阅 `props` 上的 `subscription`   并实例化 ， 通过 props 传递给 子组件 ， 以此类推 

这样保证了 组件的 更新顺序