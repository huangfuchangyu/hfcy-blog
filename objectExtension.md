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

