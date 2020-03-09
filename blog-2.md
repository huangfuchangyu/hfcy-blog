## 函数





### 简述



> 函数就是对象，其表现如下
>
> > 函数可以在运行时动态创建，还可以在程序执行过程中创建。
> >
> > 函数可以分配给变量，可以将他们的引用复制给其他变量，可以被扩展，此外，除少数情况外，函数还可以被删除
> >
> > 可以作为参数传递给其他函数，并且还可以被其他函数返回
> >
> > 函数可以有自己的属性和方法



> 函数提供了作用域， 在js 中没有{} 定义局部作用域，也就是说 块 并不创建作用域，
>
> 因此，如果在条件 if 条件语句  或者 for 以及 while  循环中，使用var 创建的变量，
>
> 并不意味着对于 if 或 for 是局部变量， 它仅对于包装函数来说是局部变量，如果没有
>
> 包装函数，他将成为一个全局变量



``` javascript
// 命名函数表达式
var add = function add(a, b) {
    return a + b
}

// 函数表达式，又称匿名函数
var add = function(a, b) {
    return a + b
}
```

> 当省略了第二个add 并且以一个未命名的函数表达式作为结束，并不会影响到该函数的定义以及后续的调用，唯一的区别在于该函数对象的name 属性将会成为一个空字符串，name 属性是 JavaScript属性的一个扩展（并不是ECMA 标准的一部分），除特殊情况有用， 如 ： firebug调试  或者 递归的情况



* 函数的提升

  > 对于所有的变量，无论在函数体的何处进行声明，都会在后台被提升到函数的顶部，而这对于函数同样适用，其原因在于函数只是分配给变量的对象，当适用函数声明时，函数定义也被提升，而不仅仅是函数声明被提升

  ``` javascript
  function foo() {
      alert('floba foo')
  }
  
  function bar() {
      alert('flobal bar')
  }
  
  function hoistMe() {
      console.log(typeof foo)    // function
      console.log(typeof bar)    // undefined
      
      foo()  // 'local foo'
      bar()  // TypeError : bar is not a function
      
      // 函数声明 
      // 变量 foo 及其实现被提升
      function foo() {
          alert('local foo')
      }
      
      // 函数表达式
      // 仅 变量 bar 被提升
      // 函数实现并未被提升
      var bar = function() {
          alert('local bar')
      }
  }
  ```

  > javascript 函数的两个特性
  >
  > > 他们都是对象
  > >
  > > 提供局部作用域



### 回调模式

### 回调与作用域

### 异步事件监听器

### 超时

### 返回函数

### 自定义函数

### 即时函数

* 优点和用法

  > 即时函数定义的变量将会用于自调用函数，不用担心全局空间被临时变量所污染。



### 即时对象初始化

* 保护全局作用域不受污染的另一种方法，是即时对象初始化模式，这种模式是带有init（）方法的对象，该方法在创建对象后将会立即执行，init（） 函数负责所有的初始化任务

``` javascript
({
    // 配置常数
    maxWidth: 800,
    maxHeight: 500,
    
    gimmeMax: function() {
      // TODO  
    },
    
    // 初始化
    init: function() {
        // 做一些初始化任务
    }
}).init()
```



### 初始化分支

* 初始化时分支 也称  加载时分支，是一种优化模式，当知道某个条件在整个生命周期内部不会发生改变的时候，仅对该条件执行一次

``` javascript
// 之前

var utils = {
    addListener: functin(el, type, fn) {
    
    	if(typeof window.addEventListener === 'function') {
            el.addEventListener(type, fn, false)
        }else if (typeof document.attachEvent === 'function') {
            el.attachEvent('on'+type, fn)
        }else {
            el['on'+type] = fn
        }
	
	},
    removeListener: function() {
        // 几乎一样
    }
}


// 优化后

// 接口
var utils = {
    addEventListener: null,
    removeEventListener: null
}

if(typeof window.addEventListener === 'function') {
   utils.addEventListener = function(el, type, fn) {
       el.addEventListener(type, fn, false)
   }
}else if (typeof document.attachEvent === 'function'){
    utils.addEventListener = function(el, type, fn) {
      el.attachEvent('on'+type, fn)
   }
}else{
     utils.addEventListener = function(el, type, fn) {
        el['on'+type] = fn
   }
}
```





### 函数属性----备忘模式

* 可以在任何时候将自定义属性添加到你的函数中，自定义属性的其中一个用例是缓存函数的结果，因此，在下一次调用该函数时就不用重复做潜在的繁重计算。

``` java
var myFunc = function() {
    var cacheKey = JSON.stringify([].slice.call(arguments))
    var result;
    
    if(!muFunc.cache[cacheKey]) {
        result = {}
        // 进行正常计算
        muFunc.cache[cacheKey] = result
    }
}

myFunc.cache = {}
```





### 配置对象

* 配置对象是一种提供更加整洁api 的方法，当函数的参数列表很长时适用

``` javascript
// 优化前
function addPerson(first, last, age, sex, addr) {
    
}
addPerson('bob', 'brown', 19, 'boy', 'aaaaaa')


// 优化后
function addPerson(conf) {
    let {first, last, age, sex, addr} = conf
}

var conf = {
    first: 'bob',
    last: 'brown'
    last: 19,
    sex: 'boy',
    addr: 'aaaaaa'
}
addPerson(conf)
```





### 函数应用

* 在一些纯粹的函数式编程语言中，函数并不描述为被调用（called 或 invoked），而是描述为应用(applied),在JavaScript 中 我们适用 apply 来 应用函数。 调用函数和应用函数可以得到完全相同的结果。

``` javascript
// 定义函数
var sayHi = functin(who) {
    return 'hello' + who
}

// 调用函数
sayHi('world')

// 应用函数
sayHi.apply(null, ['world'])
```





### Curry

* 当我们调用同一个函数，并且传递的参数绝大多数都是相同的，那么该函数可能是用于Curry化的一个很好的候选参数

* 使函数 理解 并 处理 部分应用的过程 就成为 curry 过程

``` javascript
  function schonfinkelize(fn) {

    var slice = [].slice
    var stored_args = slice.call(arguments, 1)

    // slice 方法可以用来将一个类数组（Array-like）对象/集合转换成一个新数组
    // Array.prototype.slice.call(arguments)
    // 你也可以简单的使用 [].slice.call(arguments)

    return function () {
      var new_args = slice.call(arguments)
      var args = stored_args.concat(new_args)

      return fn.apply(null, args)
    }
  }

  function add(x, y, z) {
    return x + y + z
  }

  var add5 = schonfinkelize(add, 5, 5)

  console.log(add5(5))
```

