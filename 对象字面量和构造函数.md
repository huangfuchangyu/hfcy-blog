## 对象字面量和构造函数

* 对象字面量

  > 字面量语法的显著优点在于他仅需要输入更短的字符
  >
  > 另一个原因在于他并没有作用于解析，因为可能以同样的名字创建了一个局部的构造函数，解释器需要从调用Object()的位置开始一直向上查询作用于链，知道发现全局的Object()构造函数



### 对象构造函数捕捉

* 将数字 字符串 以及布尔值传递到new Object() 构造函数中 ，其结果是获得了以不同构造函数所创建的对象



``` javascript
// 一个空对象 （反模式）
var o = new Object()
console.log(o.constructor === Object)    // true

// 一个数值对象
var o = new Object(1)
console.log(o.constructor === Object)    // false
console.log(o.constructor === Number)    // true
console.log(o.toFixed(2))                // 1.00

// 一个字符串对象
var o = new Object('i m a string')
console.log(o.constructor === String)    // true
// 一般的对象都没有 substring()方法
// 但是字符串对象都有该方法
cosnole.log(typeof o.substring)         // function

// 一个布尔对象
var o = new Object(true)
console.log(o.constructor === Boolean)  // true


```

> 当传递给 Object() 构造函数的值时动态的 ，并且直到运行是才能确定其类型时，Object（）构造函数可能会导致意想不到的结果 ，因此 不要使用 new Object（）构造函数  ， 应该使用更为简单，可靠的对象字面量模式



### 自定义构造函数

* 当以 new 操作符 调用构造函数时， 函数内部将会发生以下情况

> 创建一个空对象并且this变量引用了该对象，同时还继承了该函数的原型
>
> 属性和方法都被加入到this引用的对象中
>
> 新创建的对象由this所引用，并且最后隐式的返回this（如果没有显式的返回其他对象）

``` javascript
var Person = function(name) {
    var this = Object.create(Person.prototype)    
    this.name = name
    
    return this
}
```



* 作用于安全的构造函数

  > 为了避免 不使用 new 操作符 调用构造函数 而产生的作用于泄露问题 
  >
  > 可以再构造函数中检查this是否为构造函数的一个实例

  ``` javascript
  function Person() {
      
      if(!(this instanceof Person)){
         	  return new Person()
         }
      
     // ...
  }
  ```

  

  > 另一种检测实例对象的通用方法是 与 arguments.callee  相比较 而不是在代码中硬编码构造函数名

  ``` javascript
  if(!(this instanceof arguments.callee)) {
     	return new arguments.callee()
     }
  ```

  

  

  ### 数组构造函数的特殊性

  

  > 当向 Array（） 构造函数传递单个数字时， 他并不会成为第一个数组元素的值， 相反 他却确定了数组的长度

  ``` javascript
  // 具有一个元素的数组
  var a = [3]
  console.log(a.length)    // 1
  console.log(a[0])        // 3
  
  // 具有3个元素的数组
  
  var a = new Array(3)
  console.log(a.length)   // 3
  console.log(a[0])       // undefined
  ```

  

  > 使用数组字面量

  ``` javascript 
  var a = [3.14]
  console.log(a[0])       // 3.14
  
  var a = new Array(3.10) // rangeError : invalid array length
  console.log(typeof a)   // undefined
  ```

  



###     正则表达式字面量语法

		> TODO 











### 基本值类型包装器

* javascript 具有五个基本值类型  ： 数字,  布尔,  字符串,  null  和  undefined ,   除了  null   和  undefined 以外，其他三个具有 所谓的 基本包装对象  ， 可以使用内置的构造函数  Number()  String() Boolean() 创建包装对象

  

  >  为了说明 基本数字 和  数字对象之间的差异  请看一下例子

  ``` javascript
  // 一个基本数值
  var n = 100
  console.log(typeof n)    // number
  
  // 一个 数值对象
  var nobj = new Number(100)
  console.log(typeof nobj)     // object 
  ```

  > 包装对象包含了一些有用的属性和方法，例如： subString(), toLowerCase() 等，但是这些方法在基本只类型上也能起作用，只要调用这些方法，基本只类型就可以在后台被临时转换成一个对象，并且表现得犹如一个对象
  >
  > 通常使用包装对象的原因之一是可以可以扩充值以及持久保存状态的需要，这是由于基本值类型并不是对象，不可能扩充属性

  ``` javascript
  // 基本值类型
  var greet = 'hello there'
  
  // 为了使用split() 方法， 基本值类型被转换成对象
  greet.split(' ')     // hello
  
  // 增加一个原始数据类型并不会报错
  greet.smile = true
  
  // 但是并不会实际生效
  typeof greet.smile     // undefined
  ```

  > 此处 greet  只能临时转换成对象，使得该属性/方法可访问，且运行是不会报错，另一方面 ， 如果 使用 new String() 来定义一个对象，那么对其扩充的smile 属性会按照预期运行。 