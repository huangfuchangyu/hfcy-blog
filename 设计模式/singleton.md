## 单例模式

> 单例模式的思想在于保证一个特定的类只有一个实例，当第二次用一个类创建对象的时候，应该得到与第一次创建对象完全相同的对象。



### 使用 new 操作符

* 预期行为

  ``` javascript
  var uni = new Universe()
  var uni2 = new Universe()
  
  uni === uni2   // true
  ```

  

* 可以使用全局变量来存储实例，但是并不推荐这种写法，因为全局变量是有缺点的
* 也可以在构造函数的静态属性中缓存实例，这是一种很好的实现方法，这种简洁的语法唯一的缺点在于instance属性时公开的，在外部代码也可能回修改该属性
* 可以将该实现实例包装再闭包中，这样可以保证该实例的私有性并且可以保证不会被构造函数之外的代码所修改，其代价是带来了额外的闭包开销



### 静态属性中的实例

``` javascript
function Universe() {
    
    if(typeof Universe.instance === 'object') {
       return Universe,instance
    }
    
    this.start_time = 0
    this.bang = 'big'
    
    Universe.instance = this
    
    //隐式返回
    // return this
}

// 测试
var uni = new Universe()
var uni2 = new Universe()

uni === uni2   // true
```



### 闭包中的实例

``` javascript
function Universe() {
    var instance = this
    
    this.start_time = 0
    this.bang = 'big'
    
    Universe = function() {
        return instance
    }
}

// 测试
var uni = new Universe()
var uni2 = new Universe()

uni === uni2    // true
```

> 当第一次调用原始构造函数时，它像往常一样返回this,
>
> 在第二次，第三次调用时，它将执行重写构造函数

> 这种重写构造函数的缺点在于 ，会丢失所有在初始定义和重定义时刻之间添加到他里面的属性

``` javascript
Universe.prototype.nothing = true

var uni = new Universe()

Universe.prototype.everything = true

var uni2 = new Universe()


// 测试
uni.nothing    // true
uni2.nothing  // true

uni.everything  // undefined
uni2.everything  // undefined

uni.constructor.name   // Universe
uni.constructor === Universe   // false  因为 Universe 在 第二次 被调用的时候被重写了 虽然名字相同， 同理 prototype 也被重写

```

> 如果想按照预期运行 ，需要做一些调整

``` javascript
function Universe() {
    var instance
    
    Universe = function() {
        return instance
    }
    
    Universe.prototype = this
    
    instance = new Universe()
    
    instance.constructor = Universe
    
    instance.start_time = 0
    instance.bang = 'big'
    
    return instance
}
```

> 另一种解决方案也是将构造函数和实例包装在即使函数中

``` javascript
var Universe

(
    function(){
        var isntance
        
        Universe = function universe() {
            
            if(instance) return instance
            
            instance = this
            
            instance.start_time = 0
            instance.bang = 'big'
        }
    }()
)
```

