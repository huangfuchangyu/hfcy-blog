## call 的模拟实现

> call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法



先来看一下 call 的用法

``` javascript
function foo() {
    console.log(this.val)
}

let obj = {
    val: 5
}

foo.call(obj)  // 5
```

思路 ：  

如果能把 上面的代码转换成这种形式 

``` javascript
let obj = {
    val: 5,
    foo
}

obj.foo()    // 5
```



开始冻手实现

``` javascript
function foo() {
    console.log(this.val)
}

let obj = {
    val: 5
}

Function.prototype.myCall = function(context) {
    // this 为 调用 myCall 的方法
    context.fn = this
    context.fn()
    
    delete context.fn
}

foo.myCall(obj)    // 5
```



还有一种场景 ， 当 foo() 方法 可以传参数

``` javascript
function foo(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.val)
}

let obj = {
    val: 5
}
```

实现方案(虽然用es6 去实现es3 的 call 有点那啥 ，     但是    也行吧。。。)

``` javascript
Function.prototype.myCall = function(context) {
    
    let argsArr = Array.from(arguments)
  
    let params = argsArr.slice(1)
    
    // this 为 调用 myCall 的方法
    context.fn = this
    context.fn(...params)
    
    delete context.fn
}

bar.myCall(obj, 'bob', 18)     // bob  18  15
```



还有两个小点 需要补充

*  this 可以传 null  当为 null 的时候  等同于 window
* 函数时可以有 返回值

``` javascript
Function.prototype.myCall = function(context) {
    let context = context || window
    let argsArr = Array.from(arguments)
  
    let params = argsArr.slice(1)
    
    // this 为 调用 myCall 的方法
    context.fn = this
    
    let res =   context.fn(...params)
    
    delete context.fn
    
    return res
}
```





apply 的实现跟 call 类似  只不过参数有区别