## Apply

> apply() 方法调用一个具有给定this 值的函数， 以及作为一个数组（或类似数组的对象）提供的参数

> call() 方法的作用和 apply()方法类似， 区别就是 call() 方法接受的是参数列表， 而 apply() 方法接受的是一个参数数组

#### 语法

``` javascript
func.apply(thisArg, [argsArray])
```

#### 参数

1.thisArg

> 必选的， 在func 函数运行时使用的this 值， 注意 ， 在非严格模式 下 ， 指定为 null 或 undefined 时 会自动替换为指向全局对象， 原始值则会包装

2.argsArray

> 可选的 ， 一个数组或者类数组对象， 其中的数组元素将作为单独的参数传递给func 函数， 如果该参数的职位  null 或 undefined ，则表示不需要传入任何参数， 从 ES5 开始可以使用类数组对象

#### 返回值

> 调用有 指定this 值 和参数的函数的结果



#### 描述

在调用一个存在的函数时， 你可以为其指定一个this 对象， this 指正在调用这个函数的对象， 使用 apply ， 可以只写一次这个方法， 然后在 另一个对象中继承它， 而不用在新对象中不断重复写该方法

apply 和 call 非常相似， 不同之处在于提供参数的方式， apply 使用参数数组而不是参数列表

可也以使用arguments 作为argsArray 参数， arguments 是函数的局部变量， 他可以被用作被调用对象的所有未指定的参数， 这样 在使用apply 函数的时候就不需要知道被调用对象的所有参数， 可以使用arguments 来把所有参数传递给被调用对象， 被调用对象接下来就负责处理这些函数

从 ECMASCRIPT5 开始，可以使用 类数组对象，就是说只要有一个length 属性， 和  （0 ... length-1）  范围的整数属性



#### 示例

* 用 apply 将数组 添加到另一个数组

``` javascript
var array = ['a', 'b']
var elements = [0,1,2]
array.push.apply(array, elements)
console.info(array)    // ['a', 'b', 0,1,2]
```

* 使用 apply 和 内置函数

``` javascript
// apply 用法允许你在某些本来需要写成遍历数组变量的任务中使用内建函数
//eg: 使用  Math.max   Math.min 来找出数组的最大值和最小值
var numbers = [1,2,3,4,5,6]

var max = Math.max.apply(null, numbers)  // 基本等同于Math.max(numbers[0],...)
var min = Math.min.apply(null, numbers)

// 对比用循环完成
max = null
min = null

for(let item of numbers) {
    if(item > max) max = item
    if(item < min) min = item
}
```

> 如果用上面的方式调用 apply ， 会有超出 JavaScript引擎的参数长度限制的风险， 当你对一个方法传入非常多的参数（比如一万个）时， 就非常有可能导致越界问题， 这个临界值是根据不同的JavaScript引擎定的（JavaScript 核心中已经做了硬编码， 参数个数限制在 65536）因为这个限制是未指定的， 有些引擎会抛出异常， 而有些引擎会直接限制传入到方法里的参数， 超出的会丢弃， 导致参数丢失， 如果参数数组非常大 ， 建议采用参数数组切块后循环传入目标方法



* 使用apply 来 链接构造器

``` javascript
// 可以使用 apply 来链接一个对象构造器， 使能够在构造器中使用一个类数组对象而非参数列表

Function.prototype.constructor = function(aArgs) {
    var oNew = Object.create(this.prototype)
    this.apply(oNew, aArgs)
    return oNew
}
```







## Bind

> bind() 方法创建一个新函数， 在 bind 被调用时， 这个新函数的this 被指定为bind的第一个参数， 而其余参数将作为新函数的参数， 供调用时使用



#### 语法

``` javascript
func.bind(thisArg, arg1, arg2, ...)
```



#### 参数

1. thisArg

   调用绑定函数时作为this 参数传递给目标函数的值， 如果使用new 运算符构造绑定函数， 则忽略改值， 当使用 bind 在 setTimeout 中创建一个函数时（作为回调函数）， 作为 thisArg 传递的任何原始值都将转换为 object ， 如果 bind 参数列表为空， 执行作用域的this 将被视为新函数的 thisArg

2. arg1, arg2, ...

   当目标函数被调用时， 被预制入绑定函数的参数列表中的函数



#### 返回值

​	返回一个原函数的拷贝， 被与置入绑定函数的参数列表中



#### 描述

​	bind() 函数会创建一个新的绑定函数（bound function, BF）绑定函数是一个  exotic function object(怪异函数对象， ES5中的术语) , 它包装了原函数对象， 调用 绑定函数 通常会导致执行 包装函数

绑定函数具有以下内部属性： 

* [[BoundTargetFunction]] - 包装的函数对象
* [[BoundThis]] - 在调用包装函数时始终作为 this 值 传递的值
* [[BoundArguments]] - 列表 ， 在对包装函数做任何调用都会优先用列表元素填充参数列表
* [[call]] - 执行与此函数关联的代码， 通过函数调用表达式调用， 内部方法的参数是一个this 值 和一个包含通过调用表达式传递给函数的参数列表



**当调用绑定函数时， 他调用 [[BoundTargetFunction]]上的内部方法[[Call]] , 就像这样 Call(BoundThis, BoundArguments)**

绑定函数也可以使用new 运算符构造， 它会表现为目标函数已经被构建完毕一样， 提供的this 值会被忽略， 当前置参数仍会提供模拟函数



#### 示例

* 创建绑定函数

  ``` javascript
  this.x = 9
  var module = {
      x: 81,
      getX: function() {return this.x}
  }
  
  module.getX()   // 81
  
  var retrieveX = module.getX
  
  retrieveX()    // 9  因为是在全局作用域调用的
  
  var boundGetX = retrieveX.bind(module)
  boundGetX()    // 81
  ```

  

* 偏函数

  ``` javascript
  // bind() 的另一个最简单的用法是使一个函数拥有预设的初始参数， 只要将这些参数
  // 作为 bind() 的参数写在 this 后面， 当绑定函数被调用时， 这些参数会被
  // 插入到目标函数的参数列表的开始位置， 传递给绑定函数的参数会跟在他们后面
  
  function list() {
      return Array.prototype.slice.call(arguments)
  }
  
  function addArguments(arg1, arg2) {
      return arg1 + arg2
  }
  
  var list1 = list(1,2,3)                   // [1,2,3]
  var result1 = addArguments(1,2)           // 3
  
  // 创建一个函数， 它拥有预设参数列表
  var leadingThirtysevenList = list.bind(null, 37)
  // 创建一个函数， 它拥有预设的第一个参数
  var addThirtyseven = addArguments.bind(null, 37)
  
  leadingThirtysevenList()    // [37]
  leadingThirtysevenList(1,2) // [37, 1,2]
  
  addThirtyseven(5)      // 37 + 5 = 42
  addThirtyseven(5, 10)  // 37 + 5 = 42  , 第二个参数会被忽略
  ```



* 配合 setTimeout 

``` javascript
// 默认情况下 使用 window.setTimeout 时 ， this 关键字会指向window 对象， 
//当类的方法中需要this 指向类的实例时， 
//就需要显示地把this 绑定到回调函数，就不会丢失该实例的引用

function LateBloomer() {
    this.patalCount = Math.ceil(Math.random() * 12) + 1
}

LateBloomer.prototype.bloom = function() {
    window.setTimeout(this.declare.bind(this), 1000)
}

LateBloomer.prototype.decalre = function() {
    console.log(`i m a beautiful flower with ${this.patalCount} petals`)
}

var flower = new LateBloomer()
flower.bloom()
```





#### 手动实现 bind

``` javascript

  // Does not work with `new funcA.bind(thisArg, args)`
  if (!Function.prototype.bind) (function () {

    var slice = Array.prototype.slice;

    Function.prototype.bind = function () {
      
      var thatFunc = this, thatArg = arguments[0];
      var args = slice.call(arguments, 1);

      if (typeof thatFunc !== 'function') {
        // closest thing possible to the ECMAScript 5
        // internal IsCallable function
        throw new TypeError('Function.prototype.bind - ' +
          'what is trying to be bound is not callable');
      }

      return function () {
        var funcArgs = args.concat(slice.call(arguments))
        return thatFunc.apply(thatArg, funcArgs);
      };

    };

  })();

```



## Call

> call() 方法使用一个指定的this 值 和单独给出的一个或多个参数来调用一个函数
>
> 该方法的语法和作用与apply() 方法类似， 只有一个区别， 就是 call（） 方法接受的是一个参数列表， 而 apply() 方法接受的是一个包含多个参数的数组



#### 语法

``` javascript
func.call(thisArg, arg1, arg2, ...)
```



#### 参数

1. thisArg

   可选的， 在 function 函数运行时使用的this 值， 注意， this 可能不是该方法看到的实际值， 如果这个函数处于 非严格模式下， 则指定为 null 或 undefined 时 会自动替换为指向全局对象， 原始值会被包装

2. arg1, arg2, ...

   指定的参数列表



#### 返回值

使用调用者提供的this 值 和 参数调用该函数的返回值， 若该方法没有返回值， 则返回 undefined



#### 描述

call() 允许为不同的对象分配和调用属于一个对象的函数/方法

call()  提供新的this 值 给当前调用的函数/ 方法， 你可以使用call 来实现继承， 写一个方法，然后让另一个对象来继承它





#### 示例

* 使用call 方法调用父构造函数

``` javascript
function Product(name, price) {
    this.name = name
    this.price = price
}

function Food(name, price) {
    Product.call(this, name, price)
    this.category = 'food'
}

function Toy(name, price) {
    Product.call(this, name, price)
    this.category = 'toy'
}

var cheese = new Food('feta', 5)'
var fun = new Toy('robot', 40)
```



* 使用 call 方法调用 匿名函数



* 使用 call 方法调用函数并制定上下文 this

  ``` javascript
  function greet() {
    var reply = [this.animal, 'typically sleep between', 				         this.sleepDuration].join(' ')
    console.log(reply)
  }
  
  var obj = {
    animal: 'cats', sleepDuration: '12 and 16 hours'
  }
  
  greet.call(obj);  // cats typically sleep between 12 and 16 hours
  
  
  // 在严格模式下，this 的值将会是 undefined。见下文
  
  'use strict';
  
  var sData = 'Wisen';
  
  function display() {
    console.log('sData value is %s ', this.sData);
  }
  
  display.call(); // Cannot read the property of 'sData' of undefined
  ```

  