## 函数



#### es6中的函数参数默认值

``` javascript
function makeRequest(url, timeout = 2000, callback = function() {}) {}
```





#### 参数默认值如何影响arguments对象

> arguments 对象会在使用参数默认值时有不同的表现，在ES5非严格模式下， arguments 对象会反映出具名参数的变化

``` javascript
function mixArgs(first, second) {
    console.log(first === arguments[0])
    console.log(second === arguments[1])
    
    first = 'c'
    second = 'd'
    
     console.log(first === arguments[0])
     console.log(second === arguments[1])
}

mixArgs('a', 'b')

// 输出 
// true
// true
// true
// true
```

> 在非严格模式下， arguments 对象总会被更新，已反映出具名参数的变化

> 在严格模式下，消除了关于 arguments 对象的这种混乱，它不在反映出具名参数的具体变化

``` javascript
function mixArgs(first, second) {
    "use strict"
    
    console.log(first === arguments[0])
    console.log(second === arguments[1])
    
    first = 'c'
    second = 'd'
    
     console.log(first === arguments[0])
     console.log(second === arguments[1])
}

mixArgs('a', 'b')

// 输出 
// true
// true
// false
// false
```

> 这一次 first 与 second 就不会在影响 arguments 对象， 输出预期结果



> 然而在使用ES6 参数默认值的函数中，无论函数是否明确运行在严格模式下，arguments 对象的表现总是会与es5 的严格模式 一致，但是有一个细节，arguments对象的使用方式发生了变化

``` javascript
// 非严格模式
function mixArgs(first, second = 'b') {
    console.log(arguments.length)
    console.log(first === arguments[0])
    console.log(second === arguments[1])
    
    first = 'c'
    second = 'd'
    
    console.log(first === arguments[0])
    console.log(second === arguments[1])
}

mixArgs('a')

// 结果
// 1
// true
// false,
// false,
// false
```

> 无论是否在 严格模式下， 改变 first 和 second 的值 不会对 arguments 对象造成影响，所以 arguments 对象始终能映射出初始调用状态







#### 参数默认值表达式

> 参数默认值最有趣的特性，或许就是默认值并不得非是基本类型值， 也可以执行一个函数来产生参数的默认值

``` javascript
function getVal() {
    return 5
}

function add(first, second = getVal()) {
    return first + second
}

console.log(add(1,1))   // 2
console.log(add(1))     // 6
```

> 仅在调用add() 函数而未提供第二个参数时， getVal() 函数才会被调用， 而在 add() 函数声明初次被解析时并不会进行调用，这意味着 getValue() 函数若被写为可变的 则默认参数获取的值可能会有变化

``` javascript
let value = 5

functoin getValue() {
    return value++
}

function add(first, second = getVlaue()) {
    return first + second
}

console.log(add(1,1))    // 2
console.log(add(1))      // 6
console.log(add(1))     // 7
```

> 还可以将前面的参数作为后面的参数的默认值

``` javascript
function add(first, second = first) {
    return first + second
}

console.log(add(1,1))   // 2
console.log(add(1))     // 2
```



> 此外 还可以将first 作为参数 传递给一个函数来产生 second 参数的值

``` javascript
function getValue(value) {
    return value + 5
}

function add(first, second = getValue(first)){
    return first + second
}
```

> 但是 仅允许引用前方的参数，前面的参数不能向后访问

``` javascript
function add(first = second, second) {
    return first + second
}

console.log(add(undefined, 1))   // 抛出错误
```







#### 参数默认值的暂时性死区

> let 与const 具有暂时性死区（TDZ）， 而参数默认值 也同样具有暂时性死区， 与let 声明相似， 函数每个参数都会创建一个新的标识符绑定， 他在初始化之前不允许被访问，否则会抛出错误， 参数初始化会在函数被调用时进行， 无论谁给参数传递了一个值，还是使用了参数的默认值

``` javascript
function getValue(value) {
    return value + 5
}

function add(first, second = getValue(first)){
    return first + second
}

console.log(add(1,1 ))  // 2
console.log(add(1))    // 7


```

> 实际上 执行了一下代码来创建first 与 second 的参数值

``` javascript
// js 调用 add（1,1） 可表示为
let first = 1
let second = 1

// js 调用 add(1)  可表示为
let first = 1
let second = getValue(first)
```

> 再看一下重写过得 add 函数 

``` javascript
function add(first = second, second) {
    return first + second
}

console.log(add(1,1))            // 2
console.log(add(undefined, 1))   // 抛出错误
```

> 此例对应着

``` javascript
// js 调用 add(1, 1) 可表示为
let first = 1
let second = 1

// js 调用 add(1 ) 可表示为
let first = second
let second = 1
```

> 函数的参数拥有各自的作用域和暂时性死区， 与函数体的作用域相分离，这意味着参数的默认值不允许访问在函数体内部声明的任意变量







#### ES5 中的不具名参数



> js 早就提供了arguments 对象 用于查看传递给函数的所有参数，这样就不必分别指定每个参数。 虽然查看 arguments 对象在大多数情况下都可以正常工作，蛋操作他仍然较为麻烦

``` javascript
function pick(object) {
    let result = Object.create(null)
    
    for(let i=1, len=arguments.length; i<len; i++) {
        result[arguments[i]] = object[arguments[i]]
    }
    
    return result
}

let book = {
    title: 'this s a book',
    auther: 'thomas',
    year: 2015
}

let bookData = pick(book, 'auther', 'year')

console.log(bookData.author)   // thomas
console.log(bookData.year)     // 2015
```

``` javascript
次函数模拟了 underscore.js 代码库的 pick（） 方法。 能够返回包含原有特定对象属性的子集副本，这个函数有两个问题， 首先 完全看不出该函数具有处理多个参数的能力， 其次， 由于第一个参数被命名直接使用， 当寻找需要复制的属性时， 必须从 arguments 对对象索引位置1 而非位置0开始处理

ES6 引入了剩余参数来解决此问题
```









#### 剩余参数

> 剩余参数（rest parameter） 由 (...) 三个点 和 紧跟着的具名参数指定， 他是包含传递给函数的其余参数的一个数组， pick() 函数可以像下面这样用剩余参数来重写

``` javascript
function(object, ...keys) {
    let result = Object.create(null)
    
    for(let i=0; len = keys.length; i++) {
        result[keysi] = object[keys[i]]
    }
    
    result result
}
```



> 函数的length 属性 用于指示具名参数的数量， 而剩余参数对其毫无影响，此例中 pick() 函数的length 属性值是 1 ， 因为只有object 参数被用于计算该值

> 注： 这种说法并不严谨，若函数 使用了默认参数， 则 length 属性不包含使用默认值的参数，并且他只能指示出第一个默认参数之前的具名参数数量, 例如 对于 

``` javascript
function example(first, second='woo', third) {} 

console.log(example.length)   // 1
```

> 函数来说， length 的值是1  而非2



* 剩余参数的限制条件

1. 函数只能有一个剩余参数， 并且必须放在最后

``` javascript
functoin pick(object, ...keys, last) {
    // 语法错误 不能在剩余参数后面使用具名参数
}
```



2. 剩余参数不能在对象字面量的setter 属性中使用

``` javascript
let object = {
    set name(...value) {
        // 语法错误 不能在setter 中使用剩余参数
    }
}
```

> 存在此限制的原因是：对象字面量的setter 被限定只能使用单个参数，而剩余参数按照定义是不限制参数数量的， 因此他在此处不被许可









#### 参数如何影响arguments对象

> 设计剩余参数是为了替代ES中的arguments对象，ES4曾经移除了arguments并添加了剩余参数，以便于允许向函数传入不限数量的参数， 尽管ES4规范被废弃，但这个想法被保持下来，并在ES6中被重新引入，不过arguments扔得以保留

> arguments对象在函数被调用时反映了传入的参数，与剩余参数能协同工作，比如 

``` javascript
checkArgs('a', 'b')

function checkArgs(...args) {
    console.log(args.length)           // 2
    console.log(arguments.length)      // 2
    console.log(args[0], arguemnts[0]) // a a
    console.log(args[1], arguemnts[1]) // b b
}
```

> arguments对象总能正确反映被传入函数的参数， 而无视剩余参数的使用





#### 函数构造器的增强

> Function 构造器允许动态创建一个新函数， 但在js中并不常用，传给构造器的参数都是字符串，他们就是目标函数的参数与函数体

``` javascript
var add = new Function('first', 'second', 'return first + second')
console.log(add(1,1))
```

> ES 6 增强了 Function 的构造器能力 ，允许使用默认参数 及剩余参数

``` javascript
var add = new Function('first', 'second=first', 'return first + second')
console.log(add(1,1))

// 剩余参数
var add = new Function('...args', 'return args[0]')
console.log(add(1,1))
```



#### 扩展运算符

``` javascript
let values = [1,2,3,4,5,6,7]

// 等价于 Math.max(1,2,3,4,5,6,7)
// 等价于 Math.max.apply(Math, values)
console.log(Math.max(...values))
```





#### ES6 的 name 属性

> 定义函数有各种各样的方式，在js 中 识别函数变得很有挑战性，此外 匿名函数表达式的流行使得调试有点困难，经常使堆栈跟踪难以被阅读和解释，为此 ES6 给所有函数 添加了name 属性

``` javascript
function doSomething() {}

var doAnotherthing = function() {}

console.log(doSomething.name)         // doSomething
console.log(doAnotherthing,name)      // doAnotherthing
```

> 匿名函数 name 属性 在 firefox 与 edge 中 仍然不支持 ， 值为 空字符串， 而 Chrome 直到 51.0 版本才提供了该特性



#### name 属性的特殊情况

``` javascript
var doSomething = function doSomethingElse(){}

var person = {
    get firstName() {return 'Bob'},
    sayName: function () {console.log(this.name)}
}

console.log(doSomething.name)     // doSomethingElse
console.log(person.sayName.name)  // sayName
```



> 函数名称还有另外两个 特殊情况 ，使用 bind() 创建的函数会在名称属性值之前带有 ‘bound' 前缀 ， 而使用 Function 构造器创建的函数 ， 其名称属性为 'anonymous'

``` javascript
var doSomething = functoin() {}

console.log(doSomething.bind().name)    'bound doSomething'
console.log((new Function()).name)       'anonymous'
```

> name 属性值未必会关联到同名变量， name 属性是为了在调试是获得有用的相关信息，所以不能用name 属性值去获取对函数的引用







#### 明确 函数的 双重用途

> 在 ES5 及更早版本中， 函数根据是否使用 new 去调用而有双重用途 ，当使用new 时，函数内部的this 是一个新对象，并作为函数的返回值

``` javascript
function Person(name) {
    this.name = name
}

var person = new Person('Bob')
var notPerson = Person('Bob')


console.log(person)      // [Object, Object]
console.log(notPerson)   // undefined
```

> 当调用Person() 来创建 notPerson 时  未使用 new 输出了 undefined 并且在非严格模式下 给 全局对象添加了 name属性  ， Person 首字母大写是指示其应当使用 new 来 调用的唯一标识， 在 js 编程中是个惯例  ,  函数双重角色的混乱情况 在 ES6 中发生了一些改变

> js 为函数 提供了两个不同的内部方法 ： [Call] 与 [Construct]  , 当 未使用 new 进行 函数调用时， [call] 方法被执行， 运行的的代码中的函数体， 而当使用  new 进行函数调用时 [Construct] 方法被执行， 负责创建一个 被称为 目标的新对象， 并且将该新目标 作为 this  去 执行 函数体 ， 拥有 [construct] 方法的 函数被称为 构造器

> 并非所有的函数都拥有 [construct] 方法 ， 因此不是所有函数都可以用 new 去调用， 后面会有介绍 箭头函数就是个例外









#### 在 ES5 中 判断 函数如何被调用

> 在 ES5 中 判断是否使用了 new 去调用 函数（即作为构造器）, 最流行的方式是使用 instanceof

```` javascript
function Person(name) {
    if(this.instanceof Person) {
       this.name = name
    }
    else{
        throw new Error('you must use new with Person')
    }
}

var person = new Person('Bob')
var notPerson = Person('Bob')
````

> 此处 对 this 的值进行了检查，来判断 其是否为构造器的一个实例， 若是， 正常继续执行， 否则 抛出错误， 这么做能奏效 是因为 【Construct】 方法 创建了Person 的一个新实例，并赋值给 this ， 但是  该方法 并不绝对可靠 ， 因为有时未使用new  但是 this 扔可能是Person 实例

``` javascript
function Person(name) {
    if(this.instanceof Person) {
       this.name = name
    }
    else{
        throw new Error('you must use new with Person')
    }
}

var person = new Person('Bob')
var notPerson = Person.call(person, 'Bob')   // 奏效了
```

> 调用 Person.call() 并将 person 变量作为第一个参数传入 ， 这意味着Person 内部的this 被设置为了 person ， 对于该函数来说 ， 没任何方法能将 这种方式与 new 调用 区分开来





#### new.target

> 为了解决这个问题 ， ES6 引入了 new.target 元属性 , 元属性指的是 ”非对象“ (例如 new 运算符) 上的属性， 并提供关联目标的附加信息， 当函数的 【construct】 方法被调用时， new 运算符的作用目标 会填入 new.target 元属性，此时函数体内部的this 值 是新创建的对象实例， 而 new.target 的值 正式该实例的构造器， 若 [call] 被执行 ， new.target 的值将会是 undefined

``` javascript
function Person(name) {
    if(typeof new.target !== 'undefined') {
       this.name = name     // 使用 new 调用
    }
    else{
        throw new Error('you must use new with Person')
    }
}

var person = new Person('Bob')
var notPerson = Person.call(person, 'Bob')   // 出错
```

> ES6 通过 新增 new.target 而消除了 函数调用方式的不确定性 









#### 块级函数

> 在 ES3 或更早的版本中， 在代码块中声明函数（即 块级函数）严格来说 应当是 一个语法错误， 但所有的浏览器却都支持该语法， 可惜每个浏览器 都有轻微的行为差异， 所以最佳实践就是切勿在代码块中声明函数， 更好的是选择使用函数表达式

> 为了孔子这种不兼容行为， ES5 的严格模式 为i代码内部的函数声明引入了一种错误

``` javascript
'use strict'
if(true) {
   function doSomething() {}   // 在 ES5 中会抛出语法错误 
 }
```

> 不过 ES6 会将 doSomething 试做块级声明， 并允许它在声明的代码块内部被访问

```` javascript
'use strict'                          // 注意 这是严格模式下

if(true) {
   console.log(typeof doSomething)    // 'function'
   function doSomething() {} 
    
   doSomething()
}

console.log(typeof doSomething)   // 'undefined'
````

> 块级函数会被提升到 代码块的顶部 ， 所以 typeof doSomething 会返回 function, 一旦 if 代码块执行完毕  ， doSomething() 也就不复存在



#### 决定何时使用块级函数

> 块级函数与 let 函数表达式相似， 在执行流跳出定义所在的代码块之后 ， 函数定义就会被移除， ， 关键区别在于 ： 
>
> > 块级函数会被提升到所在代码块的顶部， 而使用 let 函数表达式则不会

````javascript
'use strict'

if(true) {
   console.log(typeof doSomething)   // 抛出错误
    
    let doSomething = function() {
        // ...
    }
    
    doSomething()
}
````

> 此处代码在 typeof doSomething 被执行时中断了 ， 因为 let 声明 尚未被执行， doSomething() 位于 暂时性死区 ， 了解这个区别口 就能根据是否想要提升 ， 来选择 块级函数还是 let 表达式



#### 非严格模式的块级函数

> ES6 在 非严格模式下同样 允许使用 块级函数 ， 但是行为略有不同 ， 块级函数的作用域会被提升到所在函数或全局环境的顶部， 而不是代码块的顶部

```` javascript
// ES6 下

if(true) {
   console.log(typeof doSomething)   // function
    
    function doSomething() {}
    
    doSomething()
}

console.log(typeof doSomething)     // function
````

> 本例中的 doSomething() 会被提升到全局作用域， 因此 在 if 代码块外部仍然存在，ES6 标准化了这种行为， 以便移除浏览器 此前存在的不兼容行， 于是在所有ES6 运行环境中其行为都会遵循相同的方式





#### 箭头函数

> 箭头函数（arrow function） 的行为在很多重要的方面与传统的js 函数不同：

* 没有 this , super, arguments, 也没有 new.target 绑定，  this , super, arguments, 及 new.target 的值由外层最近的非箭头函数来决定
* 不能使用 new 去调用  ， 因为 箭头函数没有 【construct】 方法， 因此不能用为构造函数， 使用new 调用会抛出错误
* 没有原型， 既然不能用new 调用， 也就不需要原型， 也就是没有 prototype 属性
* 不能更改 this ， this 的值在 函数内部不能被修改， 在整个  函数的生命周期 内 其值 会保持不变
* 没有 arguments 对象， 箭头函数没有 arguments 绑定， 必须 依赖于具名函数或剩余参数来访问函数的参数
* 不允许重复的具名参数， 箭头函数不允许拥有重复的具名参数， 无论是否在严格模式下， 相对来说 ， 传统函数只有在 严格模式下才禁止这种行为

> 其余差异也聚集在减少箭头函数内部的错误与不确定性， 这样js 引擎也能更好地优化箭头函数的运行

> 注意 ： 箭头函数也拥有name 属性， 并且遵循 与其他函数相同的规则





#### 尾调用优化

> 尾调用（tail call） 指的是调用函数的语句 是另一个函数的最后语句

````javascript
function doSomething() {
    return doSomethingElse()    // 尾调用
}
````

> 在 ES5 引擎中实现的尾调用， 其处理与其他函数调用一致： 一个新的栈帧（stack frame） 被创建并推到调用栈之上， 用于表示该次函数调用，这意味着每个栈帧都被保留在内存中， 当调用栈过大时会出现问题



> ES6 在 严格模式下力图为特定的尾调用减少调用栈的大小， 非严格模式的尾调用则保持不变， 当满足以下条件时， 尾调用优化不会创建新的栈帧， 而是清除当前栈帧并再次利用它：

* 尾调用不能引用当前栈帧中的变量（意味着该函数不能是闭包）
* 进行尾调用的函数在尾调用返回结果后不能做额外操作
* 尾调用的结果作为当前函数的返回值

> 举个栗子 下面的代码满足了全部三个条件， 因此能被轻易地优化

```` javascript
'use strict'
function doSomething() {
    // 被优化
    return doSomethingElse()
}
````

> 下面的例子是无法被优化的函数

```` javascript
'use strict'
function doSomething() {
    // 未被优化 没有 return
    doSomethingElse()
}
````

> 类似的 如果在 尾调用返回结果之后 进行了额外操作 ， name该函数也无法被优化

```` javascript
'use strict'
function doSomething() {
    // 未被优化   返回后 执行了 额外的操作
    return 1+ doSomethingElse()
}
````

> 无意中 关闭了优化的另一个常见方式， 是将函数调用的结果存在一个变量之上， 之后才返回结果

``` javascript
'use strict'
function doSomething() {
    // 未被优化  调用并不在尾部
    var result = doSomethingElse()
    return result
}
```

> 使用闭包或许是需要避免的最困难的情况， 因为闭包能够访问外层作用域的变量， 会导致尾调用被关闭 





#### 如何控制尾调用优化

> 在实践中， 尾调用优化由引擎进行， 除非要尽力去优化一个函数， 否则不必对此考虑太多， 尾调用优化的主要用例是在递归函数中， 而且此时的优化能到到最大效果

```` javascript
function factorial(n) {
    if(n<=1) {
       return 1
     }
    else{
        return n * factorial(n-1)   // 未被优化 返回之后还要进行乘法
     }
}
````

> 优化后

```` javascript
function factorial(n, p=1) {
    if(n<=1) {
       return n * p
     }
    else{
        let result = n * p
        return  factorial(n-1, result)   // 被优化
     }
}
````



