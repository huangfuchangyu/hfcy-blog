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

