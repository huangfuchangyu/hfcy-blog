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
    [Symbol.replace]: function(value, replacement) {
        return value.length === 10 ? replacement : value
    },
    [Symbol.search]: function(value) {
        return value.length === 10? 0 : -1
    },
    [Symbol.split]: function(value) {
        return value.length === 10 ? ['', ''] : [value]
    }
}

let message1 = 'hello word'   // 11 characters
let message2 = 'hello john'   // 10 characters

let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10)

console.log(match1)   // null
console.log(match2)   // ['hello john']

let replace1 = message1.replace(hasLengthOf10, 'Howdy'),
    replace2 = message2.replace(hasLengthOf10, 'Howdy')

console.log(replace1)   // 'hello word'
console.log(replace2)   // 'Howdy'

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10)

console.log(search1)   // -1
console.log(search2)   // 0

let split1 = message1.spilt(hasLengthOf110),
    split2 = message2.spilt(hasLengthOf110)

console.log(split1)  // ['hello word']
console.log(split2)  // ['', '']
```

> hasLengthOf110 对象 模拟了正则表达式的工作方式， 尽管 hasLengthOf110 不是正则表达式， 但他仍然能作为参数传递给这些字符串方法， 并能够正常工作
>
> 这虽然是一个简单的例子， 却表明能够进行比现有正则表达式更复杂的匹配 让自定义模式匹配更加可行







#### Symbol.toPrimitive

> JS 经常在使用特定运算符的时候 试图 进行隐士转换， 以便将对象转换为基本类型值， 例如 当使用 ‘==’  运算符来对字符串与对象进行比较的时候， 该对象会在比较之前被转换为一个基本类型值， 到底转换为何种基本类心值， 此前属于内部操作， 而ES6 则通过 Symbol.toPrimitive 方法将其暴露出来， 以便让其可更改

> Symbol.toPrimitive 方法被定义在所有常规类型的原型上， 规定了对象在对象被转换为基本类型值的时候会发生什么， 当需要转化时， Symbol.toPrimitive 会被调用， 并按照规范传入一个提示性的字符串参数， 该参数有 3 种可能， 当参数为 ‘number’ 的时候 ， Symbol.toPrimitive 应当返回一个数值， 当参数为 ‘string’ 的时候， 应当返回一个字符串， 当参数为 ‘default ’  的时候 对返回值没有特别的要求



* 对于大部分常规对象， ‘数值模式’  依次会有下述行为
  1. 调用 valueOf() 方法， 如果结果是一个基本类型  ， 则返回它
  2. 否则 调用 toString() 方法， 如果结果是一个基本值类型， 则返回它
  3. 否则 抛出一个错误
* 对于大部分常规对象， ‘字符串模式’  会依次有下列行为 
  1. 调用 toString() 方法， 如果结果是一个基本值类型， 则返回它
  2. 否则 调用 valueOf() 方法， 如果结果是一个基本类型  ， 则返回它
  3. 否则 抛出一个错误

> 在多数情况下， 常规对象的默认模式都等价于 数值模式 （只有 Date 类型除外， 他默认使用字符串模式）  通过 定义 Symbol.toPrimitive 方法， 可以重写这些默认的转换行为

> '默认模式' 只在使用 == 运算符， + 运算符 ， 或者传递 单一参数给 Date 构造器的时候被使用， 而大部分运算符都使用字符串模式或是数值模式

``` javascript
function Temperature(degrees) {
    this.degrees = degrees
}

Temperature.prototype[Symbol.toPrimitive] = functin(hint) {
    switch(hint) {
        case 'string': 
            return this.degrees + '\u00b0'   // 温度符号
        case 'number': 
            return this.degrees
        case 'default': 
            return this.degrees + 'degrees'
    }
}

let freezing = new Temperature(32)

console.log(freezing + '!')     // '32 degrees!'
console.log(freezing / 2)       // 16
console.log(String(freezing))   // '32^'
```









#### Symbol.toStringTag

> js 最有趣的课题之一是在不同的全局执行环境中使用， 这种情况会在浏览器页面包含 iframe 是出现， 此时 页面与 iframe 页面 拥有各自的全局执行环境， 大多数情况下 ， 这并不是一个问题， 但是 当 对象已经在环境之间经历了传递， 再要识别他们的类型时， 问题就来了

> 该问题得典型例子就是 从 iframe 页面 向容器页面传递数组， 当跨域进行传递时， instanceof Array 进行检测 ， 结果却是 false,  因为该数组是由另外一个域的数组构造器创建的， 不同于当前域的数组构造器

> 面对识别数组这类问题， 可以使用 toString() 方法， 很多js 库包含了如下函数

``` javascript
function isArray(value) {
    return Object.prototype.toString.call(value) == '[object array]'
}

console.log(isArray([]))  // true
```

> ES6 通过 Symbol.toStringTag 重定义了相关行为， 该Symbol 代表了所有对象的一个属性， 定义了 Object.prototype.toString.call() 被调用时应当返回什么值

> 同样， 可以在自定义对象上定义 Symbol.toStringTag 的值

``` javascript
function Person(name) {
    this.name = name
}

Person.prorotype[Symbol.toStringTag] = 'Person'

let me = new Person('bob')

console.log(me.toString())     // '[object Person]'
console.log(Object.prototype.toString.call(me))  // '[object Person]'
```

> 本例在Person 的原型上 定义了 Symbol.toStringTag 属性， 用于给他的字符串表现形式提供默认行为， 由于 Person 的原型继承了 Object.prototype.toString() 方法  ， Symbol.toStringTag 的返回值 在调用 me.toString() 的时候也会被使用， 不过 ， 依然可以在该对象上定义自己的toString() 方法， 让他有不同的返回值， 而不影响 Object.prototype.toString.call()  

``` javascript
function Person(name) {
    this.name = name
}

Person.prorotype[Symbol.toStringTag] = 'Person'

Person.prototype.toString = function() {
    return this.name
}

let me = new Person('bob')

console.log(me.toString())     // 'bob'
console.log(Object.prototype.toString.call(me))  // '[object Person]'
```

> 由于 Person 的实例不再继承 Object.prototype.toString() 方法， 调用 me.toString() 会显示不同的结果

> 除非进行了特殊的指定， 否则所有的对象都会从Object.protytype 继承 Symbol.toStringTag 属性， 其默认的属性值是字符串 ‘Object’

> 对于开发者自定义对象， Symbol.toStringTag 的返回值不受任何限制， 例如 你可以自由使用 ‘Array' 作为 Symbol.toStringTag 属性的值

``` javascript
function Person(name) {
    this.name = name
}

Person.prorotype[Symbol.toStringTag] = 'Array'

Person.prototype.toString = function() {
    return this.name
}

let me = new Person('bob')

console.log(me.toString())     // 'bob'
console.log(Object.prototype.toString.call(me))  // '[object Array]'
```

> 在这段代码中  ， 调用 Object.prototype.toString 结果是 '[object Array]'  ， 与在真实数组上调用的结果完全一样， 这一点确实证明 Object.prototype.toString() 不再是用于识别对象类型的可靠方法

> 改变 原生对象的字符串标签也是可能的， 只需要在对象的原型上对  Symbol.toStringTag 进行赋值 

``` javascript
Array.prorotype[Symbol.toStringTag] = 'Magic'

let values = []

console.log(Object.prototype.toString.call(values))   // '[object Magic]'
```

> 尽管 尽量不要用这种方式修改内置对象， 但语言本身没有禁止该行为





