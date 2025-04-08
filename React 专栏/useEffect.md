## useEffect 精读指南



**Q:**

* 如何用 `useEffect` 模拟 `componentDidMount` 生命周期？
* 如何正确的在 `useEffect` 里请求数据？ `[]`又是什么？
* 我应该把函数当做 `effect` 的依赖吗?
* 为什么有时候会出现无限重复请求的问题？
* 为什么有时候在`effect` 里拿到的是旧的 `state`或`prop`?



**摘要Q1：**

虽然可以使用 `useEffect(fn, [])` 但他们不完全相等， 和 `componentDidMount` 不一样，`useEffect` 会捕获当次渲染的 props 和 state，所以 在回调函数里，拿到的还是初始的 props 和 state， effect 更接近于实现状态同步， 而不是响应生命周期事件



**摘要Q2**

`[]`表示 effect 没有使用任何React 数据流里的值，因此 该effect 只被调用一次是安全的，同时，`[]`也是常见问题的来源， 需要学习一些策略（主要是 `useReducer` 和 `useCallback`）来移除这些 effect 依赖， 而不是错误的忽略他们



**摘要Q3**

一版建议把不依赖props和state的函数提到你的组件外面， 并把那些仅被effect使用的函数放到 effect 里面，如果你的effect需要用到组件内的函数（包括props 传递进来的函数）， 可以在定义他们的地方用`useCallback`包一层， 这样就确保了 函数不随渲染而改变，除非自身依赖发生了改变



**摘要Q4**

略。。。



**摘要Q5**

effect 拿到的 总是 定义它的那次渲染中的 props 和 state （Capture Value 特性）



#### 每次render 都有自己的 Props  与 State

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  
  return(
  	<div>
      <p>you clicked {count} times</p>
      <button onClick={() => setCount(count+1)}></button>
    </div>
  )
}
```

Jsx 中 count 会监听状态的变化并自动更新吗？count 仅是个数字而已， 不是 `data bounding`, `watcher`, `proxy`或者其他的东西，就像

``` tex
// 组件的第1 次渲染

const count = 0
// ...
<p>you clicked {count} times<p/>


// 组件的第2 次渲染

const count = 1
// ...
<p>you clicked {count} times<p/>


// 组件的第3 次渲染

const count = 2
// ...
<p>you clicked {count} times<p/>
```

仅仅是在渲染输出中插入count 这个数字， 他由React 提供，当 `setCount` 的时候，React 会带着不同的 count 再次调用组件，然后 React 会更新DOM 以保证和渲染输出一致，**在组件的每一次渲染中，count 值 独立于其他渲染**



#### 每次render 都有自己的事件处理

``` jsx
function Counter() {
  const [count, setCount] = useState(0)
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('you clicked on:', count)
    }, 3000)
  }
  
  return(
  	<div>
      <p>you clicked {count} times</p>
      <button onClick={() => setCount(count+1)}>click me</button>
      <button onClick={handleAlertClick}>show alert</button>
    </div>
  )
}
```



[![HzsAVx.gif](https://s4.ax1x.com/2022/02/22/HzsAVx.gif)](https://imgtu.com/i/HzsAVx)

结果： 3     , 相当于 

```jsx
// 第1次渲染
function Counter() {
  const count = 0
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('you clicked on:', count)
    }, 3000)
  }
  
  return(
  	<div>
      <p>you clicked {count} times</p>
      <button onClick={() => setCount(count+1)}>click me</button>
      <button onClick={handleAlertClick}>show alert</button>
    </div>
  )
}

// 第2次渲染
function Counter() {
  const count = 1
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('you clicked on:', count)
    }, 3000)
  }
  
  return(
  	<div>
      ...
    </div>
  )
}

// 第3次渲染
function Counter() {
  const count = 2
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('you clicked on:', count)
    }, 3000)
  }
  
  return(
  	<div>
      ...
    </div>
  )
}
```

由此可见， 事件处理函数属于某一次特定的渲染，当点击的时候，他会使用那次渲染中的counter 的值（**这种特性也被称为 capture value**）



#### 每次render 都有自己的effects

同理 ， effect 也和上面的部分一样

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  
  useEffect(() => {
    document.title = `you clicked ${count} times`
  })
  
  return(
  	<div>
      <p>you clicked {count} times</p>
      <button onClick={() => setCount(count+1)}>click me</button>
    </div>
  )
}
```

Effect 是如何读取到最新的 count 的值？

我们已经知道，count 实际上就是每次渲染中的一个常量，事件处理 函数拿到的是属于它声明那次特定渲染中的 count 的常量值， 对于 effect 也同样如此

并不是 count 值 在 不变的 effect 中发生了变化， 而是  effect 函数本身在每次渲染中都不相同， 每次渲染中的 effect 拿到的都是当时那次渲染的count 的值

``` jsx
// 第 1次渲染
function Counter() {
  useEffect(() => {
    document.title = `you clicked ${0} times`
  })
}

// 第 2次渲染
function Counter() {
  useEffect(() => {
    document.title = `you clicked ${1} times`
  })
}

// 第 3次渲染
function Counter() {
  useEffect(() => {
    document.title = `you clicked ${2} times`
  })
}
```

React 会记住你提供的effect 函数， 并且会在每次更改作用于DOM 并让浏览器绘制屏幕后调用它

**所以 虽然我们说的是一个 effect ，但其实 每次渲染都是一个不同的函数，并且每个 effect 函数 得到的 props 和 state 都来自于它属于的那次特定的渲染**



有时可能想在 effect 里读取最新的值而不是 捕获的值， 最简单的实现方法是 ref



#### effects 中的 clear 

有些 effect 可能需要一个 clear 以消除副作用, 比如 消息订阅

```javascript
useEffect(() => {
  MyEvent.subMsg(props.id, handleChange)
  
  return () => {
    MyEvent.unSubMsg(props.id, handleChange)
  }
})
```

假设 第一次渲染的时候 props.id 是 10，  第二次 渲染的时候  props.id 是 20， 你可能会认为代码的执行是这样的：

--------------------------------------------------------



* React 渲染 id = 10 的 UI 
* React  运行 id = 10 的 effect
* React 执行clear id=10 的 effect



* React 渲染 id = 20 的 UI 
* React  运行 id = 20 的 effect
* React 执行clear id=20 的 effect



**事实并不是这样的， 真正的执行顺序是**

* React 渲染 id = 10 的 UI 

* React  运行 id = 10 的 effect
* React 渲染 id = 20 的 UI 
* React  运行 id = 20 的 effect
* React 执行clear id=10 的 effect
* React 执行clear id=20 的 effect

React 只会在 浏览器绘制 后 运行effect , 这使得你的应用流畅因为大多数effect 并不会阻塞屏幕的更新， effect 的 clear 同样被延迟了 





#### 关于依赖项 不要对 effect 撒谎

几乎所有人都这么写过

``` jsx
function SearchResults() {
  function fetchData() {
    // ...
  }
  
  useEffect(() => {
    fetchData()
  }, [])
}
```

有时 我们只是想在挂载的时候运行它，如果设置了依赖项，effect 中用到的所有组件内的值都要包含在依赖中， 包括 props , state, 函数 等

但是这样做可能会引起无线请求的问题，或者 socket 会 被频繁的创建的问题， 解决问题的方案不是移除依赖项，很快 我们会了解具体的解决方案



``` jsx
functioin Counter() {
  const [count, setCount] = useState(0)
  
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1)
    }, 1000)
    
    return () => {
      clearInterval(id)
    }
  })
  
  return (
  	<h1>{count}</h1>
  )
}
```

然而 这个例子 只会递增1次  

sandBox:   https://codesandbox.io/s/mystifying-brattain-gy9sml?file=/src/index.js



第一次渲染中， count 是 0， 因此 `setCount(count + 1)` 在第一次渲染中等价于 `setCount(0 + 1)` 既然我们设置了 `[]` 依赖， effect 不会重新执行， 它后面的每一秒 都会调用`setCount(0 + 1)`

我们对 React 撒谎 说我们的effect 不依赖 组件内的任何值， 可实际上我们的effect 有依赖



**两种解决办法**

1. 诚实的告诉React 依赖策略

``` jsx
functioin Counter() {
  const [count, setCount] = useState(0)
  
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1)
    }, 1000)
    
    return () => {
      clearInterval(id)
    }
  }, [count])
  
  return (
  	<h1>{count}</h1>
  )
}
```

虽然可以解决问题 ， 但是会频繁的 创建定时器和回收定时器， 这不是我们想要的结果



2. 利用 setState的 函数形式

```jsx
useEffect(() => {
    const id = setInterval(() => {
      setCount(count => count + 1)
    }, 1000)
    
    return () => {
      clearInterval(id)
    }
  }, [])
```



但是 这种形式 也并不完美， 如果遇到下面这种情况：

```jsx
functioin Counter() {
  const [count, setCount] = useState(0)
  const [setp, setStep] = useState(2)
  
  useEffect(() => {
    const id = setInterval(() => {
      setCount(count => count + setp)
    }, 1000)
    
    return () => {
      clearInterval(id)
    }
  }, [setp])
  
  return (
  	<h1>{count}</h1>
  )
}
```

虽然，这种方式 可以得到预期的结果， 但是 如果我们不想有任何依赖， 利用`useReducer`

我们利用 `dispatch` 依赖去替换effect 的 step 结果



``` jsx
function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)
	const {count, step} = state || {}

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({type: 'tick'})
    }, 1000)

    return () => clearInterval(id)
  }, [dispatch])
}
// ---------------------------
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

**React 会保证 `dispatch` 在组件的生命周期内 保持不变 **

如果， 我们想`dispatch` 的依赖其他状态， 可以把 reducer 放到 组件内

``` jsx
function Counter({step}) {
  const [state, dispatch] = useReducer(reducer, 0)
  
  function reducer(state, action) {
    if(action.type === 'tick') {
      return state + step
    }
  }
  
  useEffect(() => {
    const id = setInterval(() => {
      dispatch({type: 'tick'})
    }, 1000)

    return () => clearInterval(id)
  }, [dispatch])
}
```

但是 这种模式会使一些优化时效， 应该避免滥用它





#### 把函数移动到 effect 里

一个典型的误解是认为 函数不应该称为依赖，举个例子 下面的代码看上去可以运行正常

``` jsx
function SearchResults() {
  const [data, setData] = useState({hits: []})
  
  async function fetchData() {
    const reuslt = await axios('url')
    setData(reuslt.data)
  }
  
  useEffect(() => {
    fetchData()
  }, [])
}
```

需要明确的是 ， 上面的代码可以正常工作，但是这样的组件在 日渐复杂的迭代的过程中我们很难保证它在各种情况下正常运行， 比如在某些函数内使用了是 state 或者 prop

``` jsx
function SearchResults() {
  const [data, setData] = useState({hits: []})
  const [query, setQuery] = useState('/aabb')
  
  async function fetchData() {
    const reuslt = await axios('url' + query)
    setData(reuslt.data)
  }
  
  useEffect(() => {
    fetchData()
  }, [])
}
```

如果我们忘记去更新使用这些函数的effect 依赖， 我们的 effects 就不会同步 props 和 state 带来的变更

``` jsx
function SearchResults() {
  const [data, setData] = useState({hits: []})
  const [query, setQuery] = useState('/aabb')
  
  async function fetchData() {
    const reuslt = await axios('url' + query)
    setData(reuslt.data)
  }
  
  useEffect(() => {
    fetchData()
  }, [query]) // deps are OK
}
```

我们可以把函数放到 依赖里吗 ？ 显然是可以的

``` jsx
function SearchResults () {
  const [query, setQuery] = useState('/aabb')
  
  const getFetchUrl = useCallback(
    () => {
      return await axios('url' + query)
    }, 
    [query]
  )
  
  useEffect(
    () => {
      getFetchUrl()
      // ...
    }, 
    [getFetchUrl]
  ) 
}
```



#### 说说竟态

下面是一个典型的 在 calss 组件里发生请求的例子

``` jsx
class Article extends Component {
  state = {
    article: null
  }
  componentDidMount() {
    this.fetchData(this.props.id)
  }
  componentDidUpdate(prevProps) {
    if(prevProps.id !== this.props.id) {
      this.fetchData(this.props.id)
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id)
    this.setData({article})
  }
}
```

这里问题的原因是请求返回的结果并不能保证顺序，比如 现请求了 id = 20,  然后更新到id = 10, 但是 id = 20 的请求并不会保证优先返回， 可以通过增加一个 状态来解决

``` jsx
function Article({id}) {
  const [article, setArticle] = useState(null)
  
  useEffect(
  	() => {
      let didCancel = false
      
      async function fetchData() {
        const article = await API.fetchArticle(id)
        if(!didCancel) {
          setArticle(article)
        }
      }
      
      fetchData()
      
      return () => {
        didCancel = true
      }
    },
    [id]
  )
}
```

