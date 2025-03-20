## this 全面解析



#### 调用位置

在理解this 的绑定过程之前， 先理解调用位置， 调用位置 就是函数在代码中被调用的位置（而不是声明的位置）通常来说 寻找调用位置就是寻找函数被调用的位置，但是做起来并没有这么简单， 因为某些编程模式可能会隐藏真正的调用位置， 最重要的是要分析调用栈， 我们关心的调用位置就在当前正在执行的函数的前一个调用中

> 下面来看一下到底什么是调用栈和调用位置  

``` javascript
function baz() {
    // 当前调用栈是 baz 
    // 因此 当前调用位置是全局作用域
    console.log('baz')
    bar()
}

function bar() {
    // 当前调用栈是  baz -> bar
    // 因此 当前调用位置是在  baz 中
    console.log('bar')
    foo()
}

function foo() {
    // 当前调用栈是 baz -> bar -> foo
    // 因此当前调用位置在 bar 中
    console.log('foo')
}

baz()  // baz 的调用位置
```



#### 绑定规则

我们来看看在函数的执行过程中调用位置如何决定this的绑定对象

你必须找到调用位置， 然后判断需要应用下面四条规则中的哪一条， 我们首先会分别解释这四条规则， 然后 多条规则都可用时他们的优先级如何排列



#### 默认绑定

最常用的函数调用类型： 独立函数调用， 可以把这条规则看作是无法应用其他规则时的默认规则

``` javascript
function foo() {
    console.log(this.a)
}

var a = 2

foo()   // 2
```

> 在代码中， foo() 是直接使用不带任何修饰的函数引用进行调用的， 因此只能使用默认绑定， 无法应用其他规则

如果是严格模式， 那么全局对象将无法使用默认绑定， 因此 this 会绑定到 undefined 

``` javascript
function foo() {
    'use strict'
    console.log(this.a)
}

var a = 2

foo()   // TypeError: this is undefined
```

> 这里有一个非常重要的细节， 虽然this 的绑定规则完全取决调用位置， 但是只有 foo 运行在 非严格模式下时 ， 默认绑定才能绑定到全局对象 ， 严格模式下与 foo 的调用位置无关

``` javascript
function foo() {
    console.log(this.a)
}

var a = 2

(function() {
    'use strict'
    foo()    // 2
})()
```









#### 隐式绑定

另一条需要考虑的规则是调用位置是否有上下文对象， 或者说是否被某个对象拥有或者包含

``` javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 2,
    foo
}

obj.foo() // 2
```

> 当函数引用上有上下文对象时， 隐式绑定规则会把函数调用中的this 绑定到这个上下文对象， 因为调用 foo 时 ， this 被绑定到obj  ， 因此 this.a  和 obj.a 是一样的

对象属性引用链中只有最顶层或者最后一层会影响调用位置

``` javascript
function foo() {
    console.log(this.a)
}

var obj2 = {
    a: 42,
    foo
}

var obj1 = {
    a: 2,
    obj2
}

obj1.obj2.foo() // 42
```





* 隐式丢失

  一个最常见的this 绑定问题就是被隐式绑定的函数会丢失绑定对象， 也就是说他会应用默认绑定， 从而把 this 绑定到全局对象上 或者 undefined 上， 取决于是否是严格模式

``` javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 2,
    foo
}

var bar = obj.foo
var a = 'oops'

bar()    // 'oops'
```

> 虽然 bar 是 obj.foo 的一个引用， 但实际上， 他引用的是 foo 函数本身， 因此 此时的bar 其实是一个不带任何修饰的函数调用， 因此应用了默认绑定



在回调函数中也会存在隐士丢失的问题

``` javascript
function foo() {
    console.log(this.a)
}

function doFoo(fn) {
    fn()
}

var obj = {
    a: 2,
    foo
}
var a = 'oops'

doFoo(obj.foo)    // 'oops'
```

> 参数传递其实就是一种隐式赋值， 因此 我们传入函数时 也会被隐式赋值， 所以结果和上个例子一样





#### 显示绑定

``` javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 2
}

foo.call(obj)   // 2
```

> 通过 foo.call 可以在调用foo 时 强制把它的this 绑定到 obj 上

如果你传入了一个原始值（字符串， 布尔类型）来当做this 的绑定对象， 这个原始值会被转换成它的对象形式（new String(..), new Boolean(..)） 这通常被称为 ‘装箱’

> 从 this 的绑定角度来说  call  和 apply  是一样的 区别仅仅体现在参数上

可是  ， 显示绑定仍然无法解决 之前提出的丢失绑定问题



* 硬绑定

``` javascript
function foo() {
    console.log(this.a)
}

var obj = {
    a: 2
}

var bar = function () {
    foo.call(obj)
}

bar()    // 2
setTimeout(bar, 1000)   // 2
bar.call(window)   // 2 
```

> 强制把foo 的this 绑定到了 obj 无论之后如何调用函数bar  他总会手动在obj 上调用foo ， 这种绑定是一种显示的强制绑定， 因此成为 硬绑定



* API 调用的 “上下文”

第三方库的许多函数， 以及 JavaScript 语言和宿主环境中许多新的内置函数， 都提供了一个可选的参数， 通常被称为 上下文 （context） 其作用 和 bind 一样， 确保你的回调函数使用指定的this 

``` javascript
function foo(el) {
    console.log(el, this.id)
}

var obj = {
    id: 'awesome'
}

[1,2,3].forEach(foo, obj)
// 1 awesome
// 2 awesome
// 3 swesome
```







#### new 绑定

使用 new  来调用函数， 或者说发生构造函数调用时， 会自动执行下面的操作

* 创建（或者说 构造）一个全新的对象
* 这个对象会被执行[[原型]] 连接
* 这个对象会绑定到函数调用的this
* 如果函数没有返回其他对象， 那么 new 表达式中的函数会自动返回这个新对象







#### 优先级

毫无疑问 默认绑定的优先级是四条推责中最低的

隐式绑定和显示绑定那个优先级更高？

``` javascript
function foo() {
    console.log(this.a)
}

var obj1 = {
    a: 2,
    foo
}

var obj2 = {
    a: 3,
    foo
}

obj1.foo()     // 2
obj2.foo()     // 3

obj1.foo.call(obj2)   // 3
obj2.foo.call(obj1)   // 2
```

**可以看到  显示绑定的优先级更高**

现在需要判断 隐式绑定和 new 绑定的优先级

``` javascript
fuction foo(something) {
    this.a = something
}

var obj1 = {
    foo
}

var obj2 = {}

obj1.foo(2)
console.log(obj1.a)  // 2

obj1.foo.call(obj2, 3)
console.log(obj2.a)   // 3

var bar = new obj1.foo(4)
console.log(obj1.a)  // 2
conosle.log(bar.a)   // 4
```

**可以看到  new 绑定比隐式绑定的优先级高 **

那么  new 绑定 和 显示绑定谁的优先级更高

> new  和  call/apply 无法一起使用， 因为无法通过 new foo.call(obj1) 来直接进行测试， 但是可以通过硬绑定来测试他俩的优先级

``` javascript
fuction foo(something) {
    this.a = something
}

var obj1 = {}

var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a)   // 2

var baz = new bar(3)
console.log(obj1.a)     // 2
console.log(baz.a)     // 3
```

**可以看到 new 绑定比 显示绑定的优先级更高** 



#### 判断 this

1. 函数是否在new  中调用， 如果是的话绑定的是新创建的对象

   ``` javascript
   var bar = new foo()
   ```

   

2. 函数是否通过 call, apply (显示绑定) 或者硬绑定调用， 如果是的话 this 绑定的是指定的对象

   ``` javascript
   var bar = foo.call(obj2)
   ```

3. 函数是否在某个上下文对象中调用（隐式绑定）， 如果是的话， this 绑定的是那个上下文对象

   ``` javascript
   var bar = obj1.foo()
   ```

4. 如果全都不是的话，使用默认绑定





#### 绑定例外

* 被忽略的this

  如果把 null 或者 undefined 作为 this 的绑定对象 传入 call apply  ，bind 这些值在调用时会被忽略， 应用 默认绑定规则

  ``` javascript
  function foo() {
      console.log(this.a)
  }
  
  var a= 2
  
  foo.call(null)  // 2
  ```

  这是一种非常常见的做法是使用 apply 来 ‘展开’ 一个数组， 并当做参数传入一个函数 ， 类似的 bind 可以对参数进行柯里化(预先设置一些参数)

  ``` javascript
  function foo(a, b) {
      console.log(`a: ${a}  ,  b: ${b}`)
  }
  
  foo.apply(null, [2,3])    // a: 2  , b: 3
  
  // 使用bind 进行 柯里化
  var bar = foo.bind(null, 2)
  bar(3)                    // a: 2  , b: 3
  ```

  > 在 ES6 中 可以使用 ... 来 代替 apply(...) 来展开 数组   foo(...[1,2])  和 foo(1,2) 是一样的





* 间接引用

  另一个需要注意的是， 可能 有意或无意的创建一个函数的**间接引用 ** 在这种情况下 调用这个函数会应用 默认绑定规则

  ``` javascript
  function foo() {
      console.log(this.a)
  }
  
  var a= 2
  
  var o = {
      a: 3,
      foo
  }
  
  var p = {a: 4}
  
  o.foo()    // 3
  
  p.foo = o.foo()
  
  p.foo()   // 2
  ```

  

* 软绑定

  之前已经看到过， 硬绑定这种方式可以把this 强制绑定到指定的对象（除了使用 new 时），防止函数调用应用默认绑定规则， 问题在于， 硬绑定会大大降低函数的灵活性， 使用硬绑定之后无法使用隐式绑定或显示绑定来修改this  

  如果可以给默认绑定指定一个全局对象和undefined 以外的值， 那就可以实现和硬绑定相同的效果， 同时保留隐式绑定或者显示绑定修改this 的能力

  ``` javascript
  if(!Function.prototype.softBind) {
     Function.prototype.softBind = function(obj) {
         var fn = this
         var curried = [].slice.call(arguemtns, 1)
         
         var bound = function() {
             return fn.apply(
             	(!this || this === (window || global))?
                 obj: this,
                 curried.concat.apply(curried, arguemnts)
             )
         }
         
         bound.prototype = Ojbect.create(fn.prototype)
         return bound
     }
  }
  
  
  // 测试下 softBind 是否实现了 软绑定
  function foo() {
      console.log(`name: ${this.name}`)
  }
  
  var obj = {name: 'obj'}
  var obj2 = {name: 'obj2'}
  var obj3 = {name: 'obj3'}
  
  var fooObj = foo.softBind(obj)
  
  fooObj()   // name: obj
  
  obj2.foo = foo.softBind(obj)
  obj2.foo()    // name: obj2
  
  fooOjb.call(obj3)   // name: obj3
  
  setTimeout(obj2.foo, 10)   // name: obj
  ```

  





* this 词法

  之前介绍的四条规则可以包含所有正常的函数， 但是 ES6 中介绍了一种无法使用这些规则的函数 ： 箭头函数

  箭头函数并不是function 关键字定义的， 而是使用 => 定义的， 她不使用this 的四中标准规则 ， 而是根据外层（函数或者 全局）作用域来决定this

  ``` javascript
  function foo() {
      // this 继承自 foo
      return a => console.log(this.a)
  }
  
  var obj1 = {a: 2}
  var obj2 = {a: 3}
  
  var bar =  foo.call(obj1)
  bar.call(obj2)   // 2
  ```

  > foo 内部创建的箭头函数会捕获调用时 foo 的this ， 由于 foo 的this 绑定到 obj1 , bar  的 this 也会绑定到 obj1 ， 箭头函数的绑定无法修改， new 也不行