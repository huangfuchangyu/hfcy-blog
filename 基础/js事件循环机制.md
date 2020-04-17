## js 事件循环机制（浏览器）

javascript 语言的一大特点是单线程的，也就意味着所有的任务都要排队执行

js 任务分为同步和异步两种， 他们处理方式不同

* 同步任务：  直接在主线程上排队执行，只有前一个任务执行完毕，才能执行后一个任务
* 异步任务： 不进入主线程， 被放在任务队列（task queue）中， 如果有多个异步任务， 则要在任务队列中排队等待

异步执行的运行机制如下

1. 所有同步任务都在主线程上执行， 形成一个执行站（execution context stack， 也就是执行上下文栈）
2. 主线程之外， 还存在一个 任务队列（task queue），只要异步任务有了运行结果， 就会在 任务队列中放置一个事件
3. 一旦  执行上下文栈 中所有的同步任务执行完毕， 系统就会读取 任务队列， 所对应的异步任务进入 执行站， 开始执行

**主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）**

在一个线程中， 事件循环是唯一的， 但是任务队列至少有一个，其中 任务队列分为

* 宏任务（macro-task）:  包含 整体的js 代码， 定时器（setTimeout/setInterval/setImmediate）, IO操作，UI Render 等
* 微任务（micro-task） :  Promise.then() , process.nextTick  等 



事件处理过程： 

![事件处理过程](E:\workspace\hfcy-blog\基础\event-loop.jpg)



事件循环机制流程：

1. 主线程执行整体 JavaScript 代码 。 形成 执行上下文栈
2. 执行 任务队列中的  宏任务
3. 宏任务执行完毕， 处理 当前宏任务执行所产生的微任务
4. 如果微任务 执行的过程中 产生了新的微任务， 则继续执行该微任务
5. 微任务执行完毕后 ， 进入到下一次事件循环 ， 执行第2 步

![](E:\workspace\hfcy-blog\基础\browser-deom1-excute-animate.gif)

(图片 来自于 http://lynnelv.github.io/js-event-loop-browser)





练习： 

``` javascript
// 分析 下面代码的输出结果

console.log('start')

setTimeout(function() {
    console.log('setTimeout callback')
}, 0)

let myPromise = new Promise(function(resolve, reject) {
    console.log('promise constructor')
    resolve()
})

myPromise.then(function() {
    console.log('promise then')
})

console.log('end')
```



``` javascript
// 输出结果为： 

start
promise constructor
end
promise then
setTimeout callback

// 过程分析 

// 宏 1
console.log('start')

setTimeout(function() {
    // 宏2
    console.log('setTimeout callback')
}, 0)

let myPromise = new Promise(function(resolve, reject) {
    // 宏1
    console.log('promise constructor')
    resolve()
})

myPromise.then(function() {
    // 微1
    console.log('promise then')
})

// 宏1
console.log('end')
```



更多练习 建议访问 :  https://juejin.im/post/5e58c618e51d4526ed66b5cf#heading-2