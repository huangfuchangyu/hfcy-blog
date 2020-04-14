## new 的模拟实现

new 的使用方法 

``` javascript
 function BaseShape(width, height) {
    this.width = width
    this.height = height
  }

 let square = new BaseShape(10, 10)

  console.log(square.width)    // 10
  console.log(square.height)   // 10
```

如果我们想要手动实现一个 new  首先要搞清楚 当用 new 调用构造函数时  ， 发生了什么 

> 1. 创建一个空对象并且this变量引用了该对象，同时还继承了该函数的原型
> 2. 属性和方法都被加入到this引用的对象中
> 3. 新创建的对象由this所引用，并且最后隐式的返回this（如果没有显式的返回其他对象）

``` javascript
// 当 new BaseShape() 时  相当于 

 function BaseShape(width, height) {
    var this = Object.create(Person.prototype) 
     
    this.width = width
    this.height = height
     
     return this
  }
```

所以  现在可以模拟实现一个new 了 

``` javascript
function SimulateNew(){}

// 我门想实现的效果如下

  let squeare = SimulateNew(BaseShape, 15, 15)

  console.log(squeare.width)
  console.log(squeare.height)
```

SimulateNew 实现如下 

``` javascript
  function SimulateNew() {
   
    let argArr = Array.from(arguments)

    let obj = Object.create(null)
    
    // 得到构造函数
    let Consturctor = argArr[0]    
	
    // 原型连接
    obj.prototype = Consturctor.prototype
    // 执行构造函数
    let returnVal = Consturctor.apply(obj, argArr.slice(1))
	// 判断返回值
    return typeof returnVal === 'object' ? returnVal : obj
  }

  // 测试结果 

  let squeare = SimulateNew(BaseShape, 15, 15)

  console.log(squeare.width)    // 15
  console.log(squeare.height)   // 15
	
```

