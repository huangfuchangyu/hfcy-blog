

首先 看一下 redux 用法

``` javascript
import React, { useEffect } from 'react'
import Store from './redux'

function App() {

  const storeSubscribe = () => {
    Store.subscribe(
        () => console.log(Store.getState())
    )
  }

  useEffect(storeSubscribe, [])

  const onBtnClick = () => {
    Store.dispatch({ type: 'NUM_ADD' })
  }

  return (
    <div className="App" onClick={onBtnClick}>
      点击+1
    </div>
  )
}

export default App

```



其中 `Store` 对象 是从 `./redux` 中引入的 

再看 `./redux`

``` javascript
import createStore from 'redux'
import reducer from './reducer'

const store = createStore(reducer)

export default store
```

由此  就能看到  `store` 对象 是由 `createStore()` 方法 返回的 ， 参数是 `reducer`

`reducer.js` 代码如下, 是一些样板代码组成

``` javascript

const initState = {
    num: 1,
}

const reducer = (state = initState, action) => {

    switch (action.type) {

        case 'NUM_ADD':
            return { ...state, num: ++state.num }

        case 'NUM_MINUS':
            return { ...state, num: ++state.num }

        default:
            return state
    }

}

export default reducer
```



所以**Store** 就是把它们联系到一起的对象。Store 有以下职责：

- 维持应用的 state；
- 提供 `getState()`方法获取 state；
- 提供 `dispatch(action)` 方法更新 state；
- 通过 `subscribe(listener)`注册监听器;

而 `Store` 由 `createStore()` 方法返回 

所以  第一版的 `createStore()` 实现如下

``` javascript
const createStore = reducer => {

    let state
    let listeners = []

    const getState = () => {
        return state
    }

    const subscribe = cb => {
        typeof cb === 'function' &&
            listeners.push(cb)
    }

    const dispatch = action => {

        state = reducer(state, action)

        for (const listenItem of listeners) {
            listenItem(state)
        }
    }

    return {
        subscribe,
        dispatch,
        getState,
    }
}

export default createStore
```



**`combineReducers`**

把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 `createStore`方法

合并后的 reducer 可以调用各个子 reducer，并把它们返回的结果合并成一个 state 对象

用法： 

``` javascript
const comReducer = combinReducers({ key1: reducer1, key2: reducer2 })
const store = createStore(comReducer)

export default store
```

返回的结构： 

``` javascript
{
    key1: {...},   // reducer1 的state
    key2: {...}    // reducer2 的state
}
```

实现如下： 

``` javascript

function combineReducers(reducerObj) {

  const reducersKeyArr = Object.keys(reducerObj)

  return (state = {}, action) => {
    let combinState = {}

    for (let reducerKey of reducersKeyArr) {

      const currentReducer = reducerObj[reducerKey]

      const preState = state[reducerKey]

      combinState[reducerKey] = currentReducer(preState, action)
    }

    return combinState

  }

}

export default combineReducers
```





下面 来看一下 `createStore` 方法 支持的第二个参数 `enhancer`

这个 store enhancer 的签名是 `createStore => createStore`，但是最简单的使用方法就是直接作为最后一个 `enhancer` 参数传递给 `createStore()`函数。



`createStore`方法 ， 支持 `enhancer` 参数后的实现如下： 

``` javascript
const createStore = (reducer, enhancer) => {
    
    if(typeof enhancer === 'function'){
       const newCreateStore = enhancer(createStore)
       return newCreateStore(reducer)
    }

	// 下面的和之前一样

    let state
    let listeners = []

    const getState = () => {
        return state
    }

    const subscribe = cb => {
        typeof cb === 'function' &&
            listeners.push(cb)
    }

    const dispatch = action => {

        state = reducer(state, action)

        for (const listenItem of listeners) {
            listenItem(state)
        }
    }

    return {
        subscribe,
        dispatch,
        getState,
    }
}

export default createStore
```



下面要说的 `applyMiddleware` 方法的返回值 就是一个 `enhancer`

所以  `applyMiddleware` 的用法为

``` javascript
let store = createStore(
  todos,
  applyMiddleware(logger)
)
```



根据`enhancer` 在 `createStore()` 中的 调用方式 ， `applyMiddleware` 方法的结构应该为

``` javascript
// let store = createStore(
//   todos,
//   applyMiddleware(logger)
// )

// if(typeof enhancer === 'function')
//    const newCreateStore = enhancer(createStore)
//    return newCreateStore(reducer)
// }

// 前面有说 ， applyMiddleware 返回值 是一个 enhancer
function applyMiddleware (middleware) {
    // enhancer 的入参 是 createStore, 返回值i也是 createStore方法
    // 所以 enhancer 方法里  还要返回一个新的 createStore 方法
    return function enhancer(createStore) {
        return function newCreateStore (reducer) {
            return createStore(reducer)
        }
    }
}
```



至此 ， `applyMiddleware` 作为 `createStore` 的 `enhancer` 只是 保证了 `createStore` 的正常运转， 还没有把 `middleware` 作为 增强

再看一下 `middleware` 中间件的结构, 以 `logger` 中间件的代码为例

``` javascript
function logger(store) {
  // next 实际为 dispatch 方法
  return function(next) {
    // 并返回一个接收 action 的新函数
    return function(action) {
        
      console.group(action.type)
      console.info('dispatching', action)
        
      let result = next(action)
      console.log('next state', store.getState())
        
      console.groupEnd()
        
      return result
        
    }
  }
}
```

每个 middleware 接受 `Store` 的 `dispatch` 和 `getState`函数作为命名参数，并返回一个函数

该函数会被传入 被称为 `next` 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数

这个函数可以直接调用 `next(action)`，或者在其他需要的时刻调用，甚至根本不去调用它



所以 结合`middleware`的结构  在 `applyMiddleware()` 中 使用 `middleware` 的方式为

``` javascript
function applyMiddleware (middleware) {
    
    return function enhancer(createStore) {
        return function newCreateStore (reducer) {
            
            const store = createStore(reducer)
            // middleware 接受 Store 作为命名参数，并返回一个函数
            const midFunc = middleware(store)
            // 该函数会被传入 被称为 `next` 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数
            const newDispatch = midFunc(store.dispatch)
            
           return {
                subscribe: store.subscribe,
                dispatch:  store.newDispatch,
                getState:  store.getState,
            }
            
        }
    }
}
```









以上

----------------------------------------------------------------------------------------------------