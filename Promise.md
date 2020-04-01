#### Promise

> Promise 是为异步操作的结果所准备的占位符， 函数可以返回一个 Promise 而不必订阅一个事件或向函数传递一个回调参数

#### Promise 生命周期

每个Promise 都会经历一个短暂的生命周期， 初始为进行态（pending state ）这表示异步尚未结束， 一个进行中的Promise 也被认为是 未处理的 （unsettled）， 一旦异步操作结束， Promise 就会被认为是已处理的（settled） 并进入到两种可能的状态之一



1. 已完成（fulfilled）： Promise 的异步操作已成功结束
2. 已拒绝（rejected ）： Promise 的异步操作未成功结束， 可能是一个错误， 或由其他原因导致

​    

内部的 [[PromiseState]] 属性会被设置为 ‘pending’ ，’fulfilled‘， 或 ’reject‘  以反映 Promise 状态， 该属性并未在 Promise 对象上 暴露出来， 因此 无法以编程的方式判断 promise 到底处于哪个状态， 不过 可以使用then() 方法 在 Promise的状态改变时 执行一些特定的操作

then 方法在所有的promise 上都存在， 并且接受两个参数

* 第一个参数时Promise 被完成时要调用的函数
* 第二个参数是Promise 被拒绝时要调用的函数



> 这种方式实现then 方法的任何对象都被成为一个 thenable 所有的 Promise 都是 thenable 



传递给 then 的两个参数都是可选的， 因此 可以监听完成与拒绝的任意组合形式

``` javascript
let promise = redFile('example.txt')

promise.then(
    function(succ) {
    	// 完成
    	console.log(succ)
	}, 
    function(err) {
    	// 拒绝
        console.log(err)
	}
)

promise.then(function() {
    // 完成
})

promise.then(null, function(err) {
    // 拒绝
})
```

> 这三个then 都操作在同一个 Promise 上
>
> > 第一个调用同时监听了完成与失败
> >
> > 第二个调用只监听了完成
> >
> > 第三个只监听了拒绝， 并不会报告成功信息







#### 创建未处理的Promise

新的Promise 使用 Promise构造器来创建， 此构造器接受单个参数: 一个被称为执行器（executor）的函数, 包含 初始化Promise 的代码， 该执行器会被传递两个名为 resolve 与 reject 的函数作为参数， resolve 用于 执行器在 成功结束时调用， reject 在 执行器操作失败后调用

``` javascript
// node.js 示例

let fs = require('fs')

function readFile(fileName) {
    return new Promise(function(resolve, reject) {
        // 异步操作
        fs.readFile(
        	fileName,
            {encoding: 'utf8'},
            function(err, contents) {
                if(err){
                    reject(err)
                    return
                }
                
                resolve(contents)
            }
        )
    })
}

let promise = readFile('example.txt')

promise.then(
	succ => {},  // 完成
    err => {}    // 拒绝
)
```

> 执行器 会在 readFile 被调用时 立即运行， 当 resolve 或 reject 在执行器内 被调用时， 一个作业被添加到作业队列中， 这杯成为作业调度（job scheduling）

``` javascript
let promise = new Promise(function(resolve, reject) {
    console.log('Promise')
    resolve()
})

promise.then(function() {
    consoel.log('resolved')
})

console.log('Hi')

// 输出结果为
Promise
Hi
resolved
```

> 产生上述输出结果是应为 完成处理函数 与 拒绝处理函数总是会在执行器的操作结束后被添加到作业队列的尾部





#### 已处理的Promise

基于Promise 执行器行为的动态本质， Promise 狗在其就是创建未处理的 Promise的最好方式， 但想让一个 Promise 代表一个已知的值， 安排一个 单纯传值给resolve（） 函数的作业并么有意义， 相反  有两种方法可使用指定的值来创建已处理的 Promise

* Promise.resolve()

``` javascript
let promise = Promise.resolve(42)

promise.then(function(val) {
    console.log(val)   // 42
})
```

* Promise.reject()

``` javascript
let promise = Promise.reject(42)

promise.catch(function(val) {
    console.log(val)   // 42
})
```









#### 非 Promise 的 thenable

Promise.resolve 与 Promise.reject 都能接受非 Promise 的thenable 作为参数

``` javascript
let thenable = {
    then: function(resolve, reject) {
        resolve(42)
    }
}

let p1 = Promise.resolve(thenable)

p1.then(functin(value) {
        console.log(value)  // 42
})
```

在此例中 Promise.resolve 调用了 thenable.then 确定了 这个 thenable 的 Promise 状态

在 ES6 之前 许多库都使用了 thenable 因此 将thenable 转换成正规的 Promise 的能力就非常重要， 能对之前已存在的库提供向下兼容， 当你不确定一个对象是否是 Promise 时， 将该对象传递给 Promise.resolve 或 Promise.reject 是最佳可行方案 ， 因为真正的Promise 只会被知己传递出来 不会被修改









#### 执行器错误

如果在执行器内部抛出了错误， 那么 Promise 的拒绝处理就会被调用

``` javascript
let promise = new Promise(function(resolve, reject) {
    throw new Error('Boom!')
})

promise.catch(err => {
    console.log(err)
})
```

> 在此例中， 执行器故意抛出了一个错误， 在每个执行器中存在隐式的 try-catch， 这个例子等价于

``` javascript
let promise = new Promise(function(resolve, reject) {
    try() {
         throw new Error('Boom!')
    }
   	catch(ex) {
        reject(ex)
    }
})
```









#### 全局的 Promise 拒绝处理

Promise 最具有争议的方面之一就是： 当一个Promise 被拒绝时 ， 若 缺少拒绝处理的函数， 就会静默失败， 由于 Promise 的本质， 并不能直观判断一个 Promise 的拒绝是否已被处理

``` javascript
let reject = Promise.reject(42)
// 在此刻 reject 不会被处理

// 一段时间后，，，
reject.catch(function(val) {
    console.log(val)  // 现在 reject 已经被处理了
})
```









#### Node.js 的拒绝处理

在 Nodejs 中  process 对象上存在两个关联到 Promise的拒绝处理的事件

* unhandledRejection: 当一个Promise 被拒绝， 而在事件循环的一个轮次中没有任何拒绝处理函数被调用， 该事件就会被触发
* rejectionHandled: 当一个 Promise 被拒绝， 并在事件循环的一个轮次之后再有拒绝处理函数被调用， 该事件就会被触发

``` javascript
let reject

Process.on('unhandledRejection', function(reason, promise) {
    console.log(reason.message)    // 'explosion'
    console.log(reject === promise)// true
})

reject = Promise.reject(new Error('explosion'))
```

