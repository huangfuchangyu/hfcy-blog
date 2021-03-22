`redux` 的 `middleware` 本质是包装了 `store` 的 `dispatch` 方法

`dispatch `方法的实现如下

``` javascript
   const dispatch = action => {

        state = reducer(state, action)

        for (const listenItem of listeners) {
            listenItem(state)
        }
    }
```



`applyMiddleware` 的 使用方法 如下： 

``` javascript
createStore(rootReducer, applyMiddleware(logger))
```

其中 `applyMiddleware` 作为 `inhancer ` 传入  这个` store enhancer` 的签名是 `createStore => createStore`

``` javascript
const createStore = (reducer, enhancer) => {
    
    if(typeof enhancer === 'function'){
       const newCreateStore = enhancer(createStore)
       return newCreateStore(reducer)
    }

    //	... 略

    return {
        subscribe,
        dispatch,
        getState,
    }
}

export default createStore
```



`applyMiddleware` 实现如下

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

对于 多个 `middle` 使用方式为 `applyMiddleware(middleware1, middleware2, middleware3)`

在 内部 使用了 `componse()`

最终转换成`componse(middleware1(middleware2(middleware3)))`

`middleware `的函数签名是 `({ getState, dispatch }) => next => action`



`next` 就是 `dispatch`   把 上一个 `middleware` 返回的 `action` 传给 下一个 `middleware`