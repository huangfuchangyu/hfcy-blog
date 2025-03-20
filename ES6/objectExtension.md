## 扩展的对象功能



#### 对象字面量语法扩展

``` javascript
function cratePerson(name, age) {
    return {
        name: name,
        age: age
    }
}
// 等价于

function cratePerson(name, age) {
    return {
        name,
        age
    }
}

// --------------------------------------

var person = {
    name: 'bob',
    sayName: function() {
        console.log(this.name)
    }
}

// 等价于

var person = {
    name: 'bob',
    sayName() {
        console.log(this.name)
    }
}
```





#### 可计算属性名

> 在 ES5 及更早期的版本中， 对象实例能使用 可计算的属性名， 只要用方括号表司法来代替 小数点表示法即可， 方括号允许你指定变量或字符串字面量为属性名， 并且在字符串中允许存在不能用于标识符的特殊字符

``` javascript
var person = {},
    lastName = 'last name'

person['first name'] = 'bob'
person[lastName] = 'kyire'

console.log(person['first name'])  // bob
console.log(person[lastName])      // kyire
```

> 此外  在 对象字面量中也可以直接使用字符串字面量作为属性

``` javascript
var person = {
    'first name': 'kyire'
}

console.log(person['first name'])   // kyire
```



> 这种模式要求 属性名事先已知， 并且能用字符串字面量表示， 然而， 若 属性名被包含在变量中， 或者必须通过计算才能获得， 那么在 ES5 中就无法为对象字面量定义这种属性



> 在 ES6 中， 可计算属性名是对象字面量语法的一部分， 也是用 方括号表示

``` javascript
var lastName = 'last name'

var person = {
    'first name': 'bob',
    [lastName] : 'kyire'
}

console.log(person['first name'])
console.log(person[lastName])
```

> 对象字面量内的方括号表面该属性名需要计算， 其结果是一个字符串， 这意味着其中可以包含表达式

``` javascript
var suffix = ' name'

var person = {
    ['first' + suffix]: 'bob',
    ['last' + suffix]: 'kyire'
}

console.log(person['first name'])    // bob
console.log(person['last name'])     // kyire
```









#### 新的方法

> ES5 开始就有这么一个设计意图： 避免创建新的全局函数， 避免在 Object 对象的原型上添加新方法， 而尽量尝试将新方法添加到合适对象上， 不过， 当新方法不适用于其他任何对象时， 全局的Object 对象就会收到越来越多的方法， ES6 也是在Object 对象上引入了两个新方法， 以便让特定任务更易完成



#### Object.is() 方法

> 在js 中 当要比较两个值时， 可能会使用 ==  或 === ， 为避免在 比较时  发生强制类型转换 ， 许多开发者偏向于后者， 但是严格相等运算符也不完全正确 ， 例如  +0 与 -0 相等，  NaN === NaN 会返回 false ， 因此 只有使用 isNaN() 函数才能正确检测 NaN



> ES6 引入了 Object.is() 方法来 弥补严格相等运算符残留的怪异缺陷， 此方法接受两个参数， 并会在二者 类型相同 并且 值 也相等是返回 true 

``` javascript
console.log( +0 == -0 )   // true
console.log( +0 === -0 )  // true
console.log(Object.is( +0, -0 ))  // false

console.log( NaN == NaN )   // false
console.log( NaN === NaN )  // false
console.log(Object.is( NaN, NaN))   // true

console.log( 5 == 5 )   // true
console.log( 5 === 5 )  // true
console.log( 5 == '5' )  // true
console.log( 5 === '5' ) // false
console.log( Object.is( 5, 5 ) ) // true
console.log( Object.is(5, '5') ) // false
```





#### Object.assign() 方法

> 混入（Mixin） 是在js 中 组合对象时最流行的模式， 再一次混入中， 一个对象会从另一个对象中接收属性与方法， 在很多 js 库中都有类似下面这样的混入方法

``` javascript
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key){
        receiver[key] = supplier[key]
    })
    
    return receiver
}
```

> 看下面的例子

``` javascript
function EventTarget() {}
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() {},
    on: function() {},
}

var myObject = {}
minxin(myObject, EventTarget.prototype)

myObject.emit('...')
```

> 此模式非常流行， 于是 ES6 也添加了Object.assign() 方法来完成同样饿行为， 该方法接收一个接受者对象， 以及任意数量的原对象， 并会返回接受者对象，  由于像前面这样的minin() 函数使用了赋值运算符（=） ， 也就无法将访问器属性复制到接受者上， 而 assign（） 更能反映出实际发生的情况

> Object.assign() 方法接受任意数量的源对象， 而接收对象会按照源对象在参数中的顺序来一次接收他们的属性， 这就可能发生后面的源对象的属性会覆盖前面的

``` javascript
var receiver = {}

Object.assign(
    receiver,
    {
        type: 'js',
        name: 'file.js'
    },
    {
        type: 'css'
    }
)

console.log(receiver.type)   // 'css'
```



> Object.assign() 不能将 源对象的访问器属性复制到接收对象中， 由于它使用了赋值运算符， 源对象的访问器属性就会转变成接收对象的数据属性

``` javascript
var receiver = {},
    
supplier = {
    get name() {
    	return "file.js"
    }
};

Object.assign(receiver, supplier);

// Object.getOwnPropertyDescriptor方法可以获取属性描述对象。
//它的第一个参数是一个对象，
//第二个参数是一个字符串，对应该对象的某个属性名。
var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");

console.log(descriptor.value); // "file.js"
console.log(descriptor.get); // undefined
```

> 在使用Object.assign() 方法时 ， supplier.name 返回的值是 file.js 于是改值就被存储到 receiver.name 数据属性上





#### 重复的对象字面量属性

> ES5 严格莫斯下， 存在重复的属性时 ， 会抛出错误

``` javascript
'use strict'

var person = {
    name: 'bob',
    name: 'bob'     // es5 严格模式下 会提示语法错误
}
```

> ES6 移除了重复属性检查， 无论是否严格模式， 当存在重复属性时 ， 排在后面的属性的值会成为该属性的实际值





#### 自有属性的枚举顺序

> ES5 并没有定义对象的枚举顺序， 而是将该问题留给了js 引擎厂商， ES6 则严格定义了对象自有属性在被枚举的返回顺序， 这影响了 Object.getOwnPropertyNames 与 Reflect.ownKeys 返回属性的方式， 也同样影响了 Object.assign() 处理 属性的顺序

> 自有属性枚举时 基本顺序如下：

1. 所有的数字类型键， 按升序排列
2. 所有的字符串类型建，按被添加的顺序排列
3. 所以的符号类型键， 也按照添加顺序

``` javascript
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
}

obj.d = 1

console.log(Object.getOwnPropertyNames(obj).join(''))   // 012acbd
```

> 对于 for-in 循环 ，并非所有的js 引擎都采用相同的处理方式，其媒体顺序扔未被确定 而Object.keys() 和 JSON.stringify（） 也使用了与 for-in 一样的枚举顺序





#### 修改对象的原型

> 对象的原型会通过构造器或 Object.create() 方法创建对象时被指定， js 编程到ES5 为止最重要的假定之一就是： 对象的原型在初始化完成后会保持不变， ES5 添加了Object.getPrototypeOf() 方法 来获取任意指定对象的原型， 不过仍然缺少在初始化之后更改原型的方法

> ES6 通过添加 Object.sePrototypeOf() 方法改变了这种假设， 此方法允许修改任意指定对象的原型， 他接受两个参数   ： 需要被修改原型的对象， 以及将会成为前者原型的对象

``` javascript
let person = {
    getGreeting() {
        return 'Hello'
    }
}

let dog = {
    getGreeting() {
        return 'woof'
    }
}

let friend = Object.create(person)   // 原型为 person  
console.log(friend.getGreeting())    // 'hello'
console.log(Object.getPrototypeOf(friend) === person)  // true

Object.setPrototypeOf(friend, dog)    // 将原型设置为 dog
console.log(friend.getGreeting())    // 'woof'
console.log(Object.getPrototypeOf(friend) === dog)  // true
```

> 对象的原型的实际值被存储在一个内部属性 【Prototype】上 ， Object.getPrototype() 方法会返回此属性存储的值， 而 Object.setPrototype() 方法则能够修改该值







#### 使用 super 引用的简单原型访问

> super 引用能更轻易的在对象原型上进行功能调用， 例如 ： 若要覆盖对象实例的一个方法， 但依然想要调用原型上的同名方法， 可以这么做

```javascript
let person = {
    getGreeting() {
        return 'Hello'
    }
}

let dog = {
    getGreeting() {
        return 'woof'
    }
}

let friend = {
    getGreeting() {
        return super.getGreeting() + 'Hi'
      //  return Object.getPrototypeOf(this).getGreeting.call(this) + 'Hi'  // 这样不是不易于尾调用优化嘛
    }
}
  
Object.setPrototypeOf(friend, person)   // 将原型设置为 person
console.log(friend.getGreeting())      // 'Hello Hi'
console.log(Object.getPrototypeOf(friend) === person)  // true

Object.setPrototypeOf(friend, dog)   // 将原型设置为 dog
console.log(friend.getGreeting())      // 'woof Hi'
console.log(Object.getPrototypeOf(friend) === dog)  // true
```

> 若要使用  super 引用来调用对象原型上的任何方法， 只要这个引用是位于 简写方法之内， 试图在简写方法之外使用 super 会导致语法错误

``` javascript
let friend = {
    getGreeting: function() {
        return super.getGreeting() + 'Hi'    // 语法错误
    }
}
```

> 此例使用了一个匿名函数作为属性， 于是调用 super.getGreeting() 就导致了语法错误 ， 因为在这种上下文中super 是不可用的

> 当使用多级继承时， Object.getPrototypeOf() 不再适用于所有场景  ， 此时， 引用就能体现出它的强大

``` javascript
let person = {
    getGreeting() {
        return 'Hello'
    }
}

let friend = {
    getGreeting() {
		return Object.getPrototypeOf(this).getGreeting.call(this) + 'Hi' 
    }
}

Object.setPrototypeOf(friend, person)    

let relative = Object.create(friend)
  
console.log(person.getGreeting())     // 'Hello'
console.log(friend.getGreeting())     // 'Hello Hi'
console.log(relative.getGreeting())   // error
```

> 调用 relative.getGreeting() 时 发生了错误 ， 因为此时 this 的值是relative 。 而 relative 的原型是 friend， 这样 friend.getGreeting().call() 调用就会导致进城开始反复进行递归调用， 直到发生堆栈错误

> 此问题在 ES5 中很难解决， 用 ES6 的 super 就很简单了

``` javascript
let person = {
    getGreeting() {
        return 'Hello'
    }
}

let friend = {
    getGreeting() {
		return super.getGreeting() + 'Hi' 
    }
}

Object.setPrototypeOf(friend, person)    

let relative = Object.create(friend)
  
console.log(person.getGreeting())     // 'Hello'
console.log(friend.getGreeting())     // 'Hello Hi'
console.log(relative.getGreeting())   // 'Hello Hi'
```









#### 正式的 “方法”  定义

> 在 ES6 之前  “方法” 的概念从未被正式定义， 他此前仅指 对象的函数属性而非数据属性， ES6 则正式将方法定义为： 一个拥有 【HomeObject】内部属性的函数， 此内部属性指向该方法所属的对象

``` javascript
let person = {
    // 方法
    getGreeting() {
        return 'Hello'
    }
}

// 并非方法
// 难道 window.shareGreeting() 不算吗？
function shareGreeting() {
    return 'Hi'
}
```

> 由于 getGreeting() 被直接赋给了一个对象， 他的 【HomeObject】 属性就是 person ， 而另一方面 ， shareGreeting() 函数被创建时， 并没有赋给一个对象， 他就不具备 【HomeObject】属性， 这种差异在大多数情况下并不重要， 然而使用 super 引用时就完全不同了

> 任何对 super 的引用 都会使用 【HomeObject】属性来判断 要做什么， 第一步是在【HomeObject】上调用 Object.getPrototypeOf() 来获取对象的原型引用， 接下来， 在该原型上查找同名函数， 最后 创建 this 绑定并调用方法

``` javascript
let person = {
    getGreeting() {
        return 'Hello'
    }
}

//原型为 person
let friend = {
    getGreeting() {
        return super.getGreeting() + 'Hi'
    }
}

Object.setPropertyOf(friend, person)
console.log(friend.getGreeting())    // 'Hello Hi'
```

> 此时 friend.getGreeting() 的 【HomeObject】值 是 friend ， 并且 friend 的原型是 person。 因此  super.getGreeting()  就等价于 person.getGreeting().call(this)