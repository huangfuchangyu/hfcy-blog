## Redux 



redux 大致分为 

* action
* reducer
* store 

几个部分



其中： 

action: 表示将要执行的动作

reducer: 该动作的执行者

store:  数据状态



#### action

``` java
// 首先看一下 action 是什么样子的

import ActionTypes from './ActionTypes'

export function addAction() {
    return {
        type: ActionTypes.ADD
    }
}
```

action 只是返回了一个对象 其中对象的key 一定是 ‘type’  看源码 action interface 是这样定义的 

``` javascript
export interface Action<T = any> {
  type: T
}
```

ActionTypes 是常量 , 表示 action 的类型

``` javascript
// ActionTypes.js

export default {
    ADD: 'add'
}
```





#### reducer

``` javascript
import ActionTypes from './ActionTypes'

const initialState = {
    num : 0
}

export default (state = initialState, action) => {
    switch(action.type) {
        case  ActionTypes.ADD: 
            return{...state, num: state.num + 1}
    }
}
```

改变状态 是在 reducer 里 ， 根据 不同的action type ， 改变不同的状态 

那如何把 action 和 reducer 联系到一起  ， 看 createStore



#### createStore()

上源码

``` javascript
export default function createStore<
  S,
  A extends Action,
  Ext = {},
  StateExt = never
>(
  reducer: Reducer<S, A>,
  preloadedState?: PreloadedState<S> | StoreEnhancer<Ext, StateExt>,
  enhancer?: StoreEnhancer<Ext, StateExt>
): Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext {

  function getState(): S {
      // ...
  }

  function subscribe(listener: () => void) {
    // ...
  }

  
  function dispatch(action: A) {
    // ...
  }


  function replaceReducer<NewState, NewActions extends A>(
    nextReducer: Reducer<NewState, NewActions>
  ): Store<ExtendState<NewState, StateExt>, NewActions, StateExt, Ext> & Ext {
   	// ...
  }


  function observable() {
    // ...
  }

  dispatch({ type: ActionTypes.INIT } as A)

  const store = ({
    dispatch: dispatch as Dispatch<A>,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  } as unknown) as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  return store
}

```

createStore 接收三个参数

* reducer

  > A function that returns the next state tree, given the current state tree and the action to handle

* preloadedState

  > The initial state

* enhancer

  > The store enhancer, You may optionally specify it to enhance the store with third-party capabilities such as middleware



其中 重点关注

**dispatch()**

看源码

``` javascript
  function dispatch(action: A) {
    // ...
    try {
      isDispatching = true
      // let currentReducer = reducer
      // reducer 为 createStore 接收的参数
      currentState = currentReducer(currentState, action)  
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

dispatch 接收一个参数  action ， 然后  把  state 和 action 传给 reducer ， 这样 就通过  dispatch（）  把  action 和 reducer 联系起来了 

此方法中还有关于 listener() 的部分 每次dispatch 都会触发 listerer的callback  关于listener 是怎么来的 见 subscribe（）



**subscribe()**

> *Adds a change listener. It will be called any time an action is dispatched*

看源码

``` javascript
  function subscribe(listener: () => void) {
   
	// ...

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      
   	  // ...
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
      currentListeners = null
    }
  }
```

这里就可以找到 dispatch 中的  listener 是从哪里来的了  同样 还有 unsubscribe





**getState()**

> *returns* *The current state tree of your application*

看源码

``` javascript
  function getState(): S {
 	// ...
	
    //  let currentState = preloadedState as S
    //  每次 dispatch 触发 ， 都会把 最新的状态 给 currentState 保存
    //  currentState 可以理解为 总是最新的状态
    return currentState as S
  }
```

很简单 只是返回了 currentState ， 所以 我们可以通过 getState（） 方法 拿到redux 里的状态