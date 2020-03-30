## Symbol

> 在 ES 已有的类型（string， number， bool, null, undefined ）之外， ES6 引入了一种新的基本类型： Symbol , Symbol 起初被设计用于创建对象私有成员， 在 Symbol 诞生之前， 将字符串作为属性名称导致可以轻易被访问,  无论使用哪种命名规范， 而 ‘私有名称’ 意味着开发者可以创建非字符串类型的属性名称， 由此可以防止使用常规手段来探查这些名称







#### 创建 Symbol 值

> Symbol 没有对象字面量形式， 这在 js  基本数据类型中是 独一无二的， 可以使用全局 Symbol 函数来创建一个Symbol值  

``` javascript
let firstName = Symbol()
let person = {}

person[firstName] = 'bob'
console.log(person[firstName])   // 'bob'
```

>  由于 Symbol 是基本类型的值 ， 因此 调用 new Symbol() 会抛出错误， 可以用过 new Object(yourSymbol) 来 创建一个符号实例， 但尚不清楚这能有什么用

> Symbol 函数还能接受一个额外的参数用于描述Symbol值， 该描述并不能访问对应的属性， 但他能用于调试

``` javascript
let firstName = Symbol('hello word')
let person = {}

person[firstName] = 'bob'

console.log('hello word' in person)   // false
console.log(person[firstName])   // 'bob'
console.log(firstName)   // 'Symbol(hello word)'
```

> Symbol 的 描述信息被存储在内部属性 [[Description]] 中， 当Symbol 的 toString() 方法被 显示或隐式调用是， 该属性就会被读取， 此外 灭有任何方法可以从代码中直接访问 【Description】 属性， 建议始终应该给Symbol 提供描述信息， 以便更好地阅读代码或跟进调试



* 识别 Symbol

  ``` javascript
  let symbol = Symbol('test symbol')
  console.log(typeof symbol)    // 'symbol'
  ```

  







#### 使用 Symbol

> 可以在 任意能使用 ‘可计算属性名’ 的场合使用 Symbol， 此前的例子已经展示了Symbol 的方括号用法， 而在对象的 ‘可计算字面量属性名’ 中也能使用Symbol， 还能在 Object.defineProperty()  或 Object.definePropertys()   调用中使用他

``` javascript
let firstName = Symbol('first name')

// 使用一个可计算字面量属性
let person = {
    [firstName]: 'bob'
}

// 属性只读
Object.defineProperty(person, firstName, {writable: false})

let lastName = Symbol('last name')

Object.definPropertys(person, {
    [lastName]: {
        value: 'kyire',
        writable: false
    }
})

console.log(person[firstName])
console.log(person[lastName])
```









#### 共享 Symbol 值

> 有时想在不同代码段中使用相同的 Symbol 值， ES6 提供了 ‘全局Symbol 注册表’  ， 使用 Symbol.for() 方法 ， 接受字符串类型的单个参数， 作为Symbol 符号值的标识符， 此参数同时也会成为该 Symbol 的描述信息

``` javascript
let uid = Symbol.for('uid')
let object = {}

object[uid] = '12345'

console.log(object[uid])  // 12345
console.log(uid)    // 'Symbol(uid)'
```

> Symbol.for 方法首先会搜索全局符号注册表， 看是否存在一个键值为 ‘uid’ 的Symbol 值， 如果有， 该方法会返回这个已存在的Symbol 值， 否则 ， 会创建一个新的 Symbol 值， 并使用该键值将其记录到全局Symbol 注册表总， 然后返回这个 新的 Symbol 值，这就意味了  使用同一个键值去调用Symbol.for() 方法都会返回同一个 Symbol 值

``` javascript
let uid = Symbol.for('uid')
let object = {}

object[uid] = '12345'

let uid2 = Symbol.for('uid')

console.log(uid === uid2)   // true
```

> 共享 Symbol 值 还有一个独特的用法， 可以使用  Symbol.keyFor() 方法在全局符号注册表中根据Symbol 值检索出对应的键值

``` javascript
let uid = Symbol.for('hello word')
console.log(Symbol.keyFor(uid))    // 'hello word'

let uid2 = Symbol.for('uid')
console.log(Symbol.keyFor(uid2))    // 'uid'

let uid3 = Symbol.for('uid')
console.log(Symbol.keyFor(uid3))    // undefined
```

> 全局 Symbol  注册表类似于 全局作用域， 是一个共享环境， 这意味着 不用当假设其中是否已经存在某些值， 在使用第三方组件时 ， 为Symbol 的键值使用命名空间能减少命名冲突的可能性， 例如 jQuery 代码应当为 他的所有键值使用  jquery.  的前缀







#### Symbol  值 的转换

> 类型转换是 js 语言重要的一部分， 能够非常灵活的将一种数据类型转换为另一种数据类型， 然而 Symbol 类型在 进行转换时非常不灵活， 因为其他类型缺乏 与 Symbol 合理的等价， 尤其是 Symbol 无法被转换为字符串或数值， 因此 将 Symbol 作为属性所达成的效果， 是其他类型无法替代的

> 之前的例子 使用了 console.log() 来展示 Symbol 值的输出 ， 这样做 自动调用了 Symbol 的 String（）方法来产生输出，  也可以手动调用 String() 方法 来获取 相同的结果

``` javascript
let uid = Symbol.for('uid'),
    desc = String(uid)

console.log(desc)   // 'Symbol(uid)'

// 如果想将 Symbol  与 字符串值拼接， 会引发错误  

desc = uid + ''    // 引发错误

// 同样 也不能把 Symbol 转换为 数值
// 也不能吧 Symbol 用于运算（加减乘除等）
```









#### 检索 Symbol 属性

> Object.keys() 与 Object.getOwnPropertyNames() 方法 可以检索对象的所有属性名称， 前者返回所有的可枚举属性名称， 而后者的返回值 则不会顾虑属性是否可枚举， 然而为了延续他们在 ES5 中的功能， 二者都不能返回Symbol 类型的属性， ES6 新增了 Object.getOwnPropertySymbols() 方法， 以便检索 对象的 Symbol 类型属性

* 前面提到的Symbol 属性默认是可枚举的， 使用 getOwnPropertyDescription() 方法去查看即可证实这一点， 原作者误认为Symbol 属性不可枚举， 后来做了修正， 其原因是因为 Object.keys() 是忽略 Symbol 属性的， for-in 循环也受到类似限制， 这些特性会对判断Symbol 属性是否可枚举造成干扰

> Object.getOwnPropertySymbols()  方法会返回一个数组， 包含了对象自有属性名中的 Symbol 值

``` javascript
let uid = Symbol.for('uid')
let object = {
    [uid]: '12345'
}

let symbols = Object.getOwnPropertySymbols(object) 

console.log(symbols.length)   // 1
console.log(symbols[0])       // 'Symbol(uid)'
console.log(object[symbols[]0]) // '12345'
```



> 所有对象起初 都不包含任何自有 Symbol 类型属性， 但 对象可以从他们的原型上继承  Symbol 类型属性， ES6 预定义了一些此类属性， 他们被称作 ‘ well-known Symbol’





#### 使用 well-known Symbol 暴露内部方法

> ES6 定义了 well-known Symbol  来代表js 中的一些公共行为， 而这些行为此前被认为只能是内部操作， 每个well-known Symbol  都对应全局 Symbol  对象的一个属性

* Symbol.hasInstance: 供 instanceof 运算符 使用的一个方法， 用于判断对象的继承关系
* Symbol.isConcatSpreadable: 一个布尔类型值， 在集合对象作为参数传递给 Array.prototype.concat() 方法时， 指示是否要将该集合的元素扁平化
* Symbol.iterator: 返回迭代器的一个方法
* Symbol.match :  供 String.prototype.match() 函数使用的一个方法， 用于比较字符串
* Symbol.replace: 供 String.prototype.replace() 方法使用， 用于替换子字符串
* Symbol.search : String.prototype.search() 方法使用， 用于定位子字符串
* Symbol.species : 用于产生派生对象的构造器
* Symbol.split : 供 string.prototype.split() 使用  ， 用于分割字符串
* Symbol.toPrimitive: 返回对象所对应的基本类型值的一个方法
* Symbol.toStringTag: 供 string.prototype.toString() 函数使用的一个方法， 用于创建对象的描述信息
* Symbol.unscopables : 一个对象， 其属性指示了哪些属性名不允许被包含在with 语句中

> 重写 well-known Symbol 所 定义的方法， 会把一个普通对象改变成奇异对象， 因为他改变了一些默认的内部行为， 这并不会对你的代码造成实质影响， 只是改变了规范描述对象的方式







#### Symbol.hasInstance 属性

> 每个函数都具有一个 Symbol.hasInstance 方法， 用于判断指定对象是否为本函数的一个实例， 这个方法定义在 Function.prototype 上， 因此所有函数都继承了 instanceof 运算符的默认行为， Symbol.hasInstance 属性自身是不可写入， 不可配置 ， 不可枚举的 ， 从而保证它不会被错误重写



> Symbol.hasInstance 方法 只接受单个参数， 即需要检测的值， 如果该值是本函数的一个实例， 则方法会返回true 

``` javascript
obj instanceof Array

// 这句代码等价于

Array[Symbol.hasInstance](obj)
```



> ES6 本质上将 instanceof 运算符 重定义为上述方法调用的简写， 这样 使用instanceof 便会触发一次方法调用， 实际上 允许你改变该运算符的工作方式

``` javascript
function MyObject() {}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(value) {
        return false
    }
})

let obj = new MyObject()

console.log(obj instanceof MyObject)   // false
```





#### Symbol.isConcatSpreadable

> js 在数组上设计了 concat() 方法用于将两个数组 连接到一起

``` javascript
let colors1 = ['red', 'blue']
let colors2 = colors1.concat( ['green', 'black'])

console.log(colors2.length)   // 4
console.log(colors2)          // ['red', 'blue', 'green', 'black']

// concat() 方法也可以接受非数组的参数， 此时这些参数只会直接被提那家到数组末尾

let colors3 = colors1.concat('green')

console.log(colors3)  // ['red', 'blue', 'green']

```

> 为何数组类型的参数 与 字符串类型的参数会被区别对待， 这是因为js 规范要求此时数组类型的参数需要被自动分离出各个子项， 而其他类型的参数无需如此处理， 在 ES6 之前 没有任何手段可以改变这种行为

> Symbol.isConcatSpreadable 属性是一个布尔类型的属性， 它表示目标对象拥有长度属性与数值类型的键， 并且数值类型键所对应的属性值在参与concat() 调用时，需要被分离为个体， 该Symbol 与其他的 well-known Symbol 不同， 默认情况下， 并不会作为任意常规对象的属性， 他只出现在特定类型对象上， 用来表示该对象在作为 concat()  参数时应如何工作， 从而有效改变该对象的默认行为， 可以用它来定义任意类型的对象， 让该对象在参与 concat()  调用时 能与数组类似

``` javascript
let collection  ={
    0: 'hello',
    1: 'word',
    length: 2,
    [Symbol.isConcatSpreadable]: true
}

let messages = ['Hi'].concat(collection)

console.log(messages.length)  // 3 
console.log(messages)   // ['Hi', 'hello', 'word']
```





#### Symbol.math,  Symbol.replace, Symbol.search 和 Symbol.split

> 在 JS 中， 字符串与正则表达式有着密切的联系， 尤其是 字符串具有几个可以接受正则表达式作为参数的方法

* match(regex) : 判断指定字符串是否与一个正则表达式相匹配；
* replace(regex, repalcement) : 对正则表达式的匹配结果进行替换;
* search(regex) : 在 字符串 内 对正则表达式的匹配结果进行定位
* split(regex) : 使用正则表达式将字符串分割成数组

> 这些与正则表达式交互的方法， 在ES6 之前的实现细节是对开发者隐藏的， 使得开发者无法将自定义对象模拟成正则表达式（并将他们传递给字符串的这些方法）， 而ES6 定义了4个符号以及对应的方法， 将内置的RegExp 对象原生行为外包出来

> 这4个 Symbol 表示将正则表达式作为字符串对应方法的第一个参数传入时应调用的方法，
>
> Symbol.match 方法 对应 match() 方法， Symbol.replace 对应 replace 方法， Symbol.search 对应 search 方法， Symbol.split 对应split()  这些Symbol 属性被定义在 RegExp.prototype 上作为默认实现， 以供对应的字符串方法使用

``` javascript

// 有效等价于 /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
        return value.length === 10? [value] : null
    },
}
```

