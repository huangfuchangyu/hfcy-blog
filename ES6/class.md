## 类



#### es5中的仿类结构

* js 在es5 及更早的版本中都不存在类， 与类 更接近的是： 创建一个构造器，然后将方法指派到该构造器的原型上，这种方式通常被称作自定义类型

  ``` javascript
  function PersonType(name) {
      this.name = name
  }
  
  PersonType.prototype.sayName = function() {
      console.log(this.name)
  }
  
  let person = new PersonType('Bob')
  person.sayName()    // 'Bob'
  
  console.log(person instanceof PersonType)  // true
  ```

  





#### 类的声明

* 类的声明以class 关键字开始，其后是 类的名称

  ``` javascript
  class PersonClass {
      // 等价于 PersonType 构造器
      constructor(name) {
          this.name = name
      }
      
      // 等价于 PersonType.prototype.sayName()
      sayName() {
          console.log(this.name)
      }
  }
  
  let person = new PersonClass('Bob')
  person.sayName()    // Bob
  ```

  > 类声明允许你在其中使用特殊的constructor 方法名称直接定义一个构造器，constructor 之外的方法名称没有特别的含义 ，因此 enjoy yourself

  > 自有属性（Own properties）： 该属性出现在实例上而不是出现在原型上，只能在类的构造器或方法内部进行创建，本例中 ， name 就是一个自有属性， 建议在构造器函数的内部创建所有可能出现的自有属性，这样在类中声明变量就会被限制在单一位置（有助于代码检查）

  > 相对于已有的自定义类型声明方式来说，类声明仅仅是以它为基础的一个语法糖





#### 为何要使用类语法

* 类声明不会被提升，函数会
* 类声明中的所有代码都会自动运行并且锁定在严格模式下
* 类的所有方法都是不可枚举的，自定义类型必须用Object,defineProperty()才可改变为不可枚举
* 类的所有方法内部没有 consturctor ，因此不可使用new 来调用他们，会抛出错误
* 调用类构造器是不使用new 会抛出错误
* 试图在累的方法内部重写类名，会抛出错误



> 这样看来，上面的PersonClass 声明实际上直接等价于一下未使用类语法的代码

``` javascript
// 直接等价于 PersonClass
let PersonType2 = (function(){
    "use strict"
    
    const PersonType2 = function() {
        
        // 作用域安全的构造函数
    	if(typeof new.target === 'undefined') {
           throw new Error('Constructor must called with new')
         }    
        
        this.name = name
    }
    
    Object.defineProperty(PersonType2.property, "sayName", {
        value: function() {
            // 函数呗调用时 不可以使用 new
            if(typeof new.target !== 'undefined') {
               throw new Error('Method must called with new')
            }
            
            console.log(this.name)
        },
        enumeranle: false,
        writable: true,
        confingureable: true
    })
    
    return PersonType2
    
}())
```

> 首先要注意这里有连个 PersonType2 声明： 一个在外部作用域的let 声明， 一个在IIFE 内部的const 声明，这说明了任何类名不能在方法内部被重写,而允许在外部重写

> 构造器函数检查了 new .target 以保证是使用 new 进行调用的

> sayName() 方法被定义为不可枚举， 此外也检查了 new.target 以保证被调用时没有使用new

* 此例说明了不使用新语法也能实现类的任何特性，只不过类语法显著简化了所有功能代码



* 不变的类名

  ``` javascript
  class Foo {
      constructor() {
          Foo = 'Bar'   // 抛出错误
      }
  }
  
  Foo = 'baz'  // 但是在类声明之后可以
  ```

  







#### 类表达式

> 类与函数相似之处在于都有两种形式： 声明与表达式

> 此处是与上例中的 PersonClass 等效的表达式

``` javascript
let PersonClass = class {
    consturcuor() {
        // ...
    }
    
    // ...
}
```





#### 具名类表达式

``` javascript
let PersonClass = class PersonClass2 {
    constructor() {
        // ...
    }
    
    // ...
}
```

> 此例中的类表达式被命名为 PersonClass2 ，PersonClass2 标识符只在自定义内部存在，在类的外部，  typeof PersonClass2 结果 为  undefined , 这是因为不存在PersonClass2 绑定









#### 作为一等公民的类

* 在编程中， 能被当做值来使用的就成为 一等公民 ，以为着他能作为参数传递给函数，能作为函数返回值，能用来给变量赋值， js 的函数就是一等公民， ES6 延续了传统，让类同样成为一等公民

``` javascript
function createObject(classDef) {
    return new calssDef()
}

let obj = createobject(
	class {
        sayHi() {
            console.log('Hi')
        }
    }
)

obj.sayHi()    // 'Hi'
```

> 此例中的 createobject() 函数 被调用时接收了一个匿名类表达式作为参数，使用new 创建了该类的一个实例， 并将其返回出来，随后 变量 obj 存储了所返回的实例



> 类表达式另一个有趣的用途是立即调用类构造器， 用于创建单例 

``` javascript
let person = new class {
    constructor(name) {
        this.name = name
    }
    
    sayName() {
        console.log(this.name)
    }
}('Bob')

person.sayName()    // 'Bob'
```









#### 访问器属性

> 自有属性需要在类构造器中创建，而 类 还允许你在原型上定义访问器属性

``` javascript
class CustomHTMLElement {
    constructor(elelment) {
        this.element = element
    }
    
    get html() {
        return this.element.innerHTML
    }
    
    set html(val) {
        this.element.innerHTML = val
    }
}

let descrtptor = Object.getOwnPrtpertyDescrtptor(CustomHTMLElement.protytype, 'html')

console.log('get' in descriptor)  // true
console.log('set' in descriptor)  // true
console.log(destriptor.enumerable)  // false

```

> 非类的等价表示如下

``` javascript
let CustomHTMLElement = (function() {
    "usr strict"
    
    const CustomHTMLElement = function(element) {
        
        // 作用域安全的构造函数
    	if(typeof new.target === 'undefined') {
           throw new Error('Constructor must called with new')
         }   
        
        this.element = element
    }
    
    Object.defineProperty(CustomHTMLElement.property, 'html', {
        enumerable: false,
        configurable: true,
        get: function() {
            return this.element.innerHTML
        },
        set: function(val) {
            this.element.innerHTML = val
        }
    })
    
    return CustomHTMLElement
    
}())
```











#### 可计算的成员名

> 类方法与类访问器属性可以使用 可计算的名称， 用 [] 包裹一个表达式

``` javascript
let methodName = 'sayName'

class PersonClass {
    constructor() {
        // ...
    }
    
    [methodName]() {
        // ...
    }
}
```



> 访问器属性也可以使用可计算的名称

``` javascript
let propertyName = 'html'

class CustomHTMLElement {
    constructor(elelment) {
        this.element = element
    }
    
    get [propertyName]() {
        return this.element.innerHTML
    }
    
    set [propertyName](val) {
        this.element.innerHTML = val
    }
}
```









#### 生成器方法

``` javascript
class MyClass {
    *createIterator() {
        yield 1
        yield 2
        yield 3
    }
}

let instance = new MyClass()
let iterator = instance.createIterator()
```









#### 静态成员

> 直接在欧股早起上添加额外的方法来模拟静态成员，这是es5 及更早版本中的一个通用模式

``` javascript
function PersonType(name) {
    this.name = name
}

// 静态方法
PersonType.create = function(name) {
    return new PersonType(name)
}

PersonType.prototype.sayName = function() {
    console.log(this.name)
}

let person = PersonType.create('Bob')
```

> 在其他编程语言中， 工厂方法PersonType.create() 会被认定为一个静态方法， 他的数据不依赖PersonType 的任何实例 ， 在 es6 中， 只要在方法或访问器名称前添加 static 属性

``` javascript
class PersonClass {
    constructor(name) {
        this.name = name
    }
    
    sayName() {
        console.log(this.name)
    }
    
    static create(name) {
        return new PersonClass(name)
    }
}

let person = new PersonClass('Bob')
```

> 静态成员不能用实例来访问，始终需要自身类名来访问







#### 使用派生类进行继承

> es6 之前 实现自定义类型的继承是个繁琐的过程 例如

``` javascript
functoin Rectangle(length, width) {
    this.length = length
    this.width = width
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width
}

function Square(length) {
    Rectangle.call(this, length, length)
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value: Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
})

let square = new Square(3)

console.log(square.getArea())  // 9
console.log(square instanceof Square)  // true
console.log(square instanceof Rectangle)  // true
```

> 等价的ES6 代码

``` javascript
class Rectangle{
    constructor(length, width) {
        this.length = length
        this.width = width
    }
    
    getArea = function() {
        return this.length * this.width
    }
}

class Square extends Rectangle {
    constructor(length) {
        // 与 Rectangle.call(this, length, length) 等价
        super(length, legnth)
    }
}

let square = new Square(3)

console.log(square.getArea())  // 9
console.log(square instanceof Square)  // true
console.log(square instanceof Rectangle)  // true
```

> 继承了其他类的类被称为 派生类， 如果派生类制定了构造器 ，就要使用super（） 否则会造成错误，
>
> 如果选择不使用构造器，super() 方法会被自动调用
>
> 例如 一下两个类是等价的

``` javascript
class Suqare extends Rectangle{
    // 没有构造器  自动调用 super（）
}

class Square extends Rectangle{
    // 存在构造器就必须调用 super（）
    constructor(...args) {
        super(...args)
    }
}
```

* 使用 super() 时 需要牢记一下几点
* 只能在派生类中使用 super（），若 尝试在非派生的类（即没有使用extends 关键字的类）或函数中使用，就会抛出错误
* 在构造器中，必须在 访问 this 之前调用 super（） 由于 super() 负责初始化this ， 因此 事先访问this 就会造成错误
* 若在类的构造器中不调用 super() 唯一避免出错的方法是在构造器中返回一个对象





#### 屏蔽类方法

> 派生类中的方法会屏蔽 基类中的同名类方法

``` javascript
class Square extends Rectangle {
    constructor(length) {
        super(length, length) 
    }
    
    // 重写并屏蔽 Rectangle.prototype.getArea()
    getArea() {
        return this.length * this.length
    }
}
```

> 如果想调用基类的方法

``` javascript
class Square extends Rectangle {
    constructor(length) {
        super(length, length) 
    }
    
    // 重写、屏蔽并调用了 Rectangle.prototype.getArea()
    getArea() {
        return super getArea()
    }
}
```









#### 继承静态成员

``` javascript
class Rectangle{
    constructor(length, width) {
        this.length = length
        this.width = width
    }
    
    getArea = function() {
        return this.length * this.width
    }
	
	static create(length, width) {
        return new Rectangle(length, width)
    }
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length) 
    }
}

let rect = Square.create(3,4)
console.log(rect instanceof Rectangle)  // true
console.log(rect instanceof Square)  // false
console.log(rect.getAra())           // 12
```







#### 从表达式中派生类

> 只要一个表达式 能够返回一个具有 construct 属性 以及 原型的函数 ， 就可以对其使用 extends

``` javascript
es6 中派生类最强大的能力，或许就是能够从表达式中派生类，只要一个表达式能够返回具有 【construct】 属性 及原型的函数， 就可以对其使用 extends
```

```javascript
function Rectangle(length, width) {
    this.length = length
    this.width = width
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length)
    }
}

let x = new Square(3)
console.log(x.getArea())   // 9
cnosole.log(x instanceof Rectangle)   // true
```

> extends 后面能接受任意的表达式， 例如能动态的决定需要继承的类

``` javascript
function Rectangle(length, width) {
    this.length = length
    this.width = width
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width
}

function getBase() {
    return Rectangle
}

class Square extends getBase() {
    constructor(length) {
        super(length, length)
    }
}

let x = new Square(3)
console.log(x.getArea())   // 9
cnosole.log(x instanceof Rectangle)   // true
```

> getBase() 函数作为类声明的一部分直接调用 ， 返回了Rectangle, 由于可以动态的决定基类，那也就能创建不同的继承方式，例如可以创建 mixin

``` javascript
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this)
    }
}

let AreaMixin  = {
    getArea() {
        return this.length * this.width
    }
}

function minxin(...mixins) {
    var base = function() {}
    Object.assign(base.prototype, ...minxins)
    
    return base
}

class Square extends minxin(AreaMixin, SerializableMixin) {
    consturctor(length) {
        super()     // 由于使用了 exetnds 关键字 所以要在构造器中调用super()
        this.length = length
        this.width = length
    }
}

let x = new Square(3)
console.log(x.getArea())   // 9
console.log(x.serialize())  // "{"length":3,"width":3}"
```



> 任意的表达式都能在extends 关键字后使用，但并非所有的表达式结果都是一个有效的类， 下列表达式类型就会明确导致错误

* null
* 生成器函数

> 试图使用为上述值的表达式来创建一个新的类实例，都会抛出错误，因为不存在[construct] 可供调用





#### 继承内置对象

> 几乎从js数组出现的那天起， 开发这就想通过继承机制来创建自己的特殊数组类型，在es5 及早期版本中，这是不可能做到的， 试图使用传统继承并不能产生功能正确的代码

``` javascript
// 内置的数组行为
var colors = []
colors[0] = 'red'
console.log(colors.length)   // 1

colors.length = 0
console.log(colors[0])   // undefined

// 在 es5 中尝试继承数组
function MyArray() {
    MyArray.apply(this, arguments)
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        name: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
})

var colors = new MyArray()
colors[0] = 'red'
console.log(colors.length)  // 0

colors.length = 0
console.log(colors[0])   // 'red'
```

> 对数组使用传统形式的js 继承， 产生了预期外的行为， MyArray 实例上的length 属性以及数值属性，其行为与内置数组并不一致，因为这些功能并未涵盖在 Array.apply 或 原型数组中



> 在 es5 的传统继承中，this 的值会先被派生类创建，随后基类构造器才被调用，这意味着this 一开始就是 MyArray的实例， 之后才使用了Array 的附加属性对其进行装饰

> 在 es6 基于类的继承中，this 的值先被基类创建， 随后才被派生类的构造器所修改，结果是this 初始就拥有作为基类的内置对象的所有功能，并能正确与之关联的所有功能



> 一下范例实际展示了基于类的特殊数组

``` javascript
class MyArray extends Array{}

var colors = new MyArray()

colors[0] = 'red'
console.log(colors.length)

colors.length = 0
console.log(colors[0]) // undefined
```





#### Symbol.species 属性

> 继承内置对象带来一个有趣的特性，能返回任意内置对象实例的方法，在派生类上却会自动返回派生类的实例，因此， 若你拥有一个继承了Array 的派生类MyArray ， 如 slice 之类的方法都会返回 MyArray 的实例

``` javascript
class MyArray extends Array{}

let items = new MyArray(1,2,3,4)
let subItems = items.slice(1,3)

console.log(items instanceof MyArray)  // true
console.log(subItems instanceof MyArray)  // true
```

> 在此代码中， slice 方法返回了 MyArray 的实例， slice 方法是从Array 上继承的，原本应当返回Array 的一个实例， 而 Symbol.species 属性在后台造成了这种变化

> Symbol.species 被用于定义一个能返回函数的静态访问器属性，每当类实例除了构造器之外的方法必须创建一个实例时， 前面返回的函数就被用为新实例的构造器， 下列内置类型都定义了Symbol.species

* Array
* ArrayBuffer
* Map
* Promise
* RegExp
* Set
* 类型化数组

> 以上每个类型都拥有默认的 Symbol.species 属性， 其返回值为this 以为着该属性总是返回自身的构造器函数, 若你准备在一个自定义类上实现此功能

``` javascript
class MyClass {
    static get [Symbol.species]() {
        return this
    }
    
    constructor(value) {
        this.value = value
    }
}
```





#### 在 类构造器中使用 new.target

> 可以通过判断 new.target 来判断 类是如何被调用的， 在简单的情况下 new,target 就等于本类的构造器函数

``` javascript
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle)
        this.length = length
        this.width = width
    }
}

var obj = new Rectangle(3,4)    // true
```

> 在 new Rectangle(3,4)  被调用时， new.target 就等于 Rectangle ， 类构造器被调用时不能缺少 new ，因此 new.target 属性 始终会在类构造器内被定义， 不过这个值并不是不变的

``` javascript
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle)
        this.length = length
        this.width = width
    }
}

class Square extends Rectangle{
    constructor(length) {
        super(length, length)
    }
}

var obj = new Square(3)    // false
```

> Square 调用了 Rectangle 构造器， 因此当 Rectangle 构造器被调用时 ， new.target等于 Square ,    可以使用  new.target 来创建一个抽象基类

``` javascript
class Shape {
    consturctor() {
        if(new.target === Shape) {
         	throw new Error('this bug cannot be instantiated directly')  
         }
    }
}

class Rectangle extends Shape {
    // ...
}

var x = new Shape     // 抛出错误
```

> 由于调用类是不能缺少 new ， 于是 new.target 属性在类构造器内部 就绝不会是 undefined