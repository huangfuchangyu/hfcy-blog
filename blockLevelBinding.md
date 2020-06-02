## 块级绑定



### var 声明与变量提升

> 使用var 关键字声明的变量，无论声明的位置在何处，都会被视为声明于所在函数的顶端（如果不在任意函数内，则视为全局作用域的顶部）这就是所谓的变量提升

``` javascript
function getValue(condition) {
    if(condition) {
     	var value = 'blue'  
    }
    else{
        // value 可以在此处访问， 值为 undefined
        return null
    }
    
    // value 可以在此处访问， 值为 undefined
}
```

value无论如何都会被创建，js引擎会将getValue函数调整为如下形式

``` javascript
function getValue(condition) {
    var value
    
    if(condition) {
     	value = 'blue'  
    }
    else{
        return null
    }
   
}
```

> value 边里的声明被提升到了函数的顶部，而初始化操作则保留在原处，这意味着else分之内value 变量也是可以访问的，在此处他的值并未被初始化，因此是undefined





### 块级声明

> 块级声明时让所有声明在指定块的作用域外都无法被访问，块级作用域（词法作用域）在如下情况被创建

* 在一个函数内部
* 在一个由一对花括号包裹的代码块内部



#### let 声明

> let 声明的语法与var的语法一致，大体上可以直接用let 代替var 进行变量声明，但会将变量的作用域限制在当前的代码块中，由于let 声明并不会被提升，因此若想让变量在整个代码块内不可用，需要手动将let声明放到代码块顶部

``` javascript
function getValue(condition) {
    if(condition) {
     let value = 'blue'  
    }
    else{
        // value 在此处不可用
        return null
    }
    // value 在此处不可用
}
```

> 由于变量value 不会被提升，因此 在if 代码块以外无法访问



#### 禁止重复声明

> 如果一个标识符已经在代码块内部被定义，那么在此代码块内使用同一个标识符声明就会导致错误

``` javascript
var count = 1

let count = 1  // 语法错误

var a = 1
var a = 2
console.log(a)    // 2  不会语法错误 会覆盖
```

> 在嵌套的作用域内使用let声明同名新变量，就不会有问题

``` javascript
var count = 1
if(condition) {
   let count = 1
}
```

> 此处的let 声明没有被抛出错误，由于它并不是在同一作用域在此创建count变量，在if 代码块内部，这个新变量会屏蔽全局的count 变量，从而在局部组织对全局count的访问



#### 常量声明

> const  声明的变量被设置后不允许再被更改，因此，所有的const 变量都需要在声明时进行初始化

``` javascript
const maxItems = 1

const maxItems  // 语法错误  未进行初始化
```

* const 声明与 let 声明一样， 都是块级声明，这意味着他们在语句块之外是无法被访问的，并且声明也不会被提升
* 另一个相似点是： 在同一作用域内定义一个已有变量是会抛出错误，无论是在全局还是局部作用域，无论该变量此前使用var 声明  还是 let 声明  

``` javascript
var message = 'hello'
let age = 17

const message = 'goodbye'   // 抛出错误
const age = 16              // 抛出错误
```



> 尽管有上述相似之处，但是let h和 const 还有一个本质区别，试图对const 声明的常量再次赋值会抛出错误，无论是否在严格模式下

``` javascript
const maxItems = 5
maxItems = 3   // 抛出错误
```



> const 声明会阻止对变量的绑定与变量自身值的修改，这意味着他不会阻止对变量成员的修改

``` javascript
const person = {
    name: 'kyire'
}

//工作正常
person.name = 'laven'

//抛出错误
person = {}
```





#### 暂时性死区

> 使用 let 或 const 声明的变量， 在到达声明位置之前都是无法访问的，试图访问会导致一个引用错误，即使所用的操作通常是安全的，例如 typeof

``` javascript
if(condition) {
 	console.log(typeof value)    // 引用错误
    let value = 123
 }
```

> 出现错误是因为 value 位于 被 js 社区 称为暂时性死区（temporal dead zone  ， TDZ）的区域内，该名称并未在ECMAScript 规范中被明确命名，但经常被描述let 或 const 声明的变量为何在声明位置之前无法被访问，

> 当js 引擎扫描接下来的代码块并发现声明变量时，他会在处理var时将声明提升到函数或者全局作用域的顶部， 而处理 let 或 const 时， 则会将声明放入暂时性死区， 任何在暂时性死区内访问变量的企图 都会导致 ‘运行时’ 错误 （runtime error）只有执行到变量的声明语句是，该变量才会从暂时性死区内被移除， 才可以安全使用

> 使用 let 和 const 声明的变量都无法规避暂时性死区



#### 循环中的块级绑定

> 当期望在for 循环内使用块级作用域时

``` javascript
for(var i=0; i<10; i++) {
    process(items[i])
}

console.log(i)   // i 在此处仍然可以被访问
```

> 如果改用let 就会看到预期行为

``` javascript
for(let i=0; i<10; i++) {
    process(items[i])
}

console.log(i)   // 抛出错误
```





#### 循环内函数

> var 的特点使得循环变量在循环作用域之外仍可以被访问，于是在循环内创建函数会产生问题

``` javascript
var funcs = []

for(var i=0; i<10; i++) {
    funcs.push(function() {console.log(i)})
}

funcs.forEach(function(func){
    func()    // 十次数值 10
})
```

> 预期这段代码会输出 0-9 的数值 ，这是因为变量i 被提升， 循环内创建的那些函数都拥有对于同一个变量i 的引用
>
> IIFE 可以修正这个问题

``` javascript
var funcs = []

for(var i=0; i<10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value)
        }
    })(i))
}

funcs.forEach(function(func){
    func()   // 0-9 依次输出
})
```



> 与var 声明 配个 IIEF 相比 ， let 能达到相同的效果

``` javascript
var funcs = []

for(let i=0; i<10; i++) {
    funcs.push(function() {console.log(i)})
}

funcs.forEach(function(func){
    func()    //  0-9 依次输出
})
```

> 这种方式 在  for-in  和 for-of 循环中同样适用





#### 循环内的常量声明

> es6 规范并没有禁止在循环中使用const 声明，但是 在循环会试图改变该变量的值时会抛出错误

``` javascript
var funcs = []

// 在一次迭代后抛出错误
for(const i=0; i<10; i++) {
    funcs.push(function() {console.log(i)})
}

```

> 另一方面 const 变量在 for-in 或 for-of 循环中使用 时 , 与 let 变量效果相同， 因此 下面代码不会导致错误

``` javascript
var funcs = [],
    obj = {
        a: true,
        b: true,
        c: true
    }

// 不会导致错误
for(const key in obj) {
    funcs.push(function() {console.log(key)})
}
```





#### 全局块级绑定

> let 与 const 不同与 var 的另一个 方面 是 在 全局作用域上的表现，当在全局作用域上使用var 时 ，他会创建一个新的 全局变量 ， 并成为 全局对象 （在浏览器中是window）的一个属性， 这意味着使用 var 可能会无意的覆盖 已有的全局属性

``` javascript
// 在浏览器中
var regExp = 'hello'
console.log(window.regExp)   // 'hello'
```

> 在全局作用域上使用 let 或 const ，虽然在全局作用域上会创建新的绑定，但是不会有任何属性呗添加到全局对象上，这就意味着不能使用let 或 const 来覆盖 一个全局变量， 智能遮蔽

``` javascript
let regExp = 'hello'
console.log(regExp)     // 'hello'
console.log(window.regExp === regExp)   // false

const ncz = 'Hi'
console.log(ncz)   // 'Hi'
console.log(ncz in window)   // false
```

