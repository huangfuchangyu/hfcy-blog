## set 与 map

> js 长期以来都只存在一种集合类型，数组类型， 但缺少更多类型的集合， 通常被当做队列与栈来使用， 数组只使用了数值型的索引，若要有非数值型的索引， 开发者便会使用非数组对象， 用他们来定制实现 set y与 map

set 是不包含重复值的列表， 一般不会像对待数组那样来访问set 中的某个项， 更常见的操作是 在 set 中检查某个值是否存在， map 则是 键与相应的值的集合， 因此 map 中的每个项都存储了两块数据， map 常被用作缓存， 存储的数据以便此后快速检索

> 以 object  来模拟 map 有以下 问题 

``` javascript
let map = Object.create(null)
map[5] = 'foo'  
map['5'] = 'bar'

console.log(map[5])     // 'bar'
```

> 由于 在 object 中 ， 数值型的键  会在内部被转成 字符串， 因此  map[5]  与 map['5']  实际上引用了同一个属性

> 如果使用对象为键 则会出现另一个问题

``` javascript
let map = Object.create(null),
    key1 = {},
    key2 = {}

map[key1] = 'foo'

console.log(map[key2])   // 'foo'
```

> 由于 key1 和 key2 被当做 键 使用  会被转成 字符串  也就是 '[object object]',   所以指向了同一个属性





#### Set 

> Set 是一种无重复值的有序列表， 允许对 Set 包含的数据进行快速访问， 从而能更有效地追踪离散值

> set 使用 new Set()  来创建，  add()  方法 能向其中添加项目，  size 属性能查看其中包含多少个项目

``` javascript
let set = new Set()
set.add(5)
set.add('5')

console.log(set.size)   // 2

```

> Set不会使用强制类型转换来判断值是否重复， 这意味着 set 可以同时包含 5  和 ‘5’  将他们作为相对独立的项存储，（在set 内部的比较 使用了  Object.is() 方法 来判断两个值是否相等， 唯一列外的事 +0  与 -0 被判定是相等的）  还可以向set 中添加对象 

``` javascript
let set = new Set()

let key1 = {}
let key2 = {}

set.add(key1)
set.add(key2)

console.log(set.size)   // 2
```

> 如果 add() 方法用相同的值进行了多次调用， 那么在第一次之后的调用会被忽略

``` javascript
let set = new Set()

set.add(5)
set.add('5')
set.add(5)     // 重复了  该调用会被忽略

console.log(set.size)   // 2
```

> 也可以使用数组来初始化 set ， 并且 set 构造器会确保不重复地使用这些值

``` javascript
let set = new Set([1,2,3,4,5,5,5])
console.log(set.size)   // 5
```

> set 构造器 实际上可以接受任意可迭代对象作为参数， 使用数组是因为他们默认就是可迭代的， set 与 map 一样， set 构造器会使用迭代器来提取参数中的值

> 可以使用 has（） 来检测 某个值是否存在与set 中

``` javascript
let set = new Set()

set.add(5)
set.add('5')

console.log(set.has(5))    // true
console.log(set.has(6))   // false
```



> 同时 也可以使用 delete() 方法移除 set 中的值， 使用 clear() 方法可清除 set 中的所有的值

``` javascript
let set = new Set()

set.add(5)
set.add('5')

console.log(set.has(5))   // true

set.delete(5)

console.log(set.has(5))   // false
console.log(set.size)    // 1

set.clear()

console.log(set.size)    // 0
```









#### set 上的  forEach() 方法

forEach() 方法会被传递给一个回调函数， 该回到函数接受三个参数

* set 中下一个位置的值
* 与第一个参数相同的值
* 目标 set 自身

> Set 版本的 forEach() 方法与数组版本有个奇怪的差异， 前者传给回调函数的前两个参数是相同的 ， 虽然看起来像是个错误， 但是这种行为却有一个正当的理由

> 具有 forEach() 方法的其他对象， 数组与Map 都会给回调函数传递三个参数， 前连个参数的位置分别是 下个位置的 值与键

> 然而 Set 没有键， ES6 标准的制定可以将Set 版本的 forEach() 方法的回调函数设定为只接受两个参数， 但这回让他不同于另外两个版本的方法， 于是为了让这些回调方法保持一致  ， 就将set 中的每一项同时认定为键与值

> 除了参数特点的差异外， 在Set 上使用forEach() 方法与 数组上基本相同

``` javascript
let set = new Set([1,2])

set.forEach(function(val, key, ownerSet) {
    console.log(val)
    console.log(key)
    console.log(ownerSet === set)
})

// 1
// 1
// true
// 2
// 2
// true
```

> 与使用数组相同， 如果想在回调函数中使用 this 可以给forEach()传入一个this 作为第二个参数

``` javascript
let set = new Set([1,2])

let processor = {
    output() {},
    process(dataSet) {
        dataSet.forEach(function() {
            // ...
        }, this)
    }
}

processor.process(set)
```





#### 将Set 转换成数组

``` javascript
let arr = [...new Set([1,2,3,4,5,5])]

console.log(arr)   // [1,2,3,4,5]
```







#### Weak Set

> 由于Set类型存储对象引用的方式， 他也可以被称为 Strong Set(强引用的 Set )  对象存储在 Set 的一个实例中， 实际上相当于 存储在变量中， 只要对于 Set 实例的引用仍然存在， 所存储的对象就无法被垃圾回收机制回收， 从而无法释放内存

``` javascript
let set = new Set()
let key = {}

set.add(key)
console.log(set.size)     // 1

key = null    // 取消原始引用

console.log(set.size)   // 1

// 重新获得原始引用
key = [...set][0]
```

> 这种结果在大部分程序中是没问题的， 但是 当js 代码在网页中运行并保持了与dom 元素的引用， 在改元素可能被其他脚本移除的情况下， 会造成 内存泄漏

> 为了缓解这个问题 ES6 引入了 Weak Set, 该类型只允许存储对象弱引用， 而不能存储基本类型的值， 对象的弱引用在他自己成为该对象的唯一引用时， 不会阻止垃圾回收









> Weak Set 使用 WeakSet 构造器来创建 ， 并包含 add() 方法， has() 方法， 以及 delete() 方法

``` javascript
let set = new WeakSet()
let key = {}

set.add(key)

console.log(set.has(key))   // true

set.delete(key)  

console.log(set.has(key))   // false
```

> 当用数组 初始化 WeakSet 时  若 数组中包含 基本类型值 ， 会抛出错误









#### Set 类型之间的关键差异

> Weak Set  与 Set 之间最大的区别是 对象的若引用

> 其他差异包括

* 对于 Weak Set 的实例， 若调用 add() 方法传入了非对象的参数， 就会抛出错误， has() 和 delete（） 则会在传入了非对象的参数时 返回  false
* Weak Set 不可迭代
* Weak Set 无法暴露出任何迭代器（例如 keys() 与 values() 方法） 因此 没有任何手段可用于判断 Weak Set 的内容

* Weak Set 没有 forEach() 方法
* Weak Set 没有 size 属性

> Weak Set 看起来功能有限， 而这对于正确管理内存是必要的， 一般来说 若指向追踪对象的引用， 应当使用 Weak Set  而不是正规 Set 









#### Map

> ES6 的  Map 类型是 键值对的有序列表 ， 键 和  值 都可以是任意类型， 键的比较 用的是  Object.is() , 这与 使用 对象来模拟 Map 截然不同 ， 因为i对象的属性会被强制转换成字符串

``` javascript
let map = new Map()

map.set('title', 'hello world')
map.set('desc', 2016)

console.log(map.get('title'))    // 'hello world'
console.log(map.get('subTitle')) // undefined
```

> 也可以将对象作为键使用， 这是之前 使用 对象来模拟 map 所无法实现的

``` javascript
let map = new Map(),
    obj1 = {},
    obj2 = {}

map.set(obj1, 5)
map.set(obj2, 11)

console.log(map.get(obj1))  // 5
console.log(map.get(obj2))  // 11
```





#### Map 的方法

> Map 和 Set 有意共享了几个方法， 允许使用相似的方式来与 Map 及 Set 进行交互， 

* has(key): 判定指定的键 是否存在于 Map 中
* delete(key) : 移除 Map 中的键以及对应的值
* clear() ： 移除 Map 中所有的键与值
* size:  用于指明包含了多少个键值对

``` javascript
let map = new Map()
map.set('name', 'bob')
map.set('age', 18)

console.log(map.size)    // 2
console.log(map.has('name'))  // true
console.log(map.get('name'))  // 'bob'

map.delete('name')
console.log(map.get('name'))  // undefined
console.log(map.size)    // 1

map.clear()
console.log(map.has('name'))  // false
console.log(map.get('name'))  // undefined
console.log(map.size)    // 0
```









#### Map 的 初始化

``` javascript
let map = new Map([['name', 'bob'], ['age', 18]])

console.log(map.has('name'))    // true
console.log(map.get('name'))    // 'bob'
```

> 虽然 由 数组 构成的 数组 看上去有点奇怪 ， 但是  由于 键可以是任意类型， 将 键 存储在数组中， 是确保他们在被添加到 Map  之前 不会被强制转换为其他类型的唯一方法





#### Map 上的 forEach() 方法

> Map 的 forEach()  方法 类似于 Set 与 数组 的同名方法， 他接受一个  三个参数的 回调函数：

* Map 中下个位置的值
* 该值所对应的的键
* 目标 Map 自身

> 回调函数的这些参数 紧密契合了数组的 forEach() 方法的行为

``` javascript
let map = new Map([['name', 'bob'], ['age', 18]])

map.forEach(function(value, key, ownerSet) {
    console.log(val)
    console.log(key)
    console.log(ownerSet)
})

// 'bob'
// 'name'
// true
// 18
// 'age'
// true
```

> 也可以给 forEach 提供第二个参数 ， 来指定回调函数中的this 值 ， 其行为 与  Set 版本的 forEach() 一致









#### Weak Map

> 在 Weak Map 中 ， 所有的键 都必须是 对象， （尝试使用非对象的键会抛出错误）， 而且这些对象都是弱引用， 不会干扰垃圾回收， 当 WeakMap 的键 在 WeakMap 之外不存在引用时， 该键会被移除

> Weak Map 的最佳使用， 就是在浏览器中创建一个 关联到特定 Dom 元素的对象， 如果 此 dom 元素 被销毁，  他就会在 Weak Map 中被自动回收

> 注意， Weak Map 的键 才是弱引用， 而值不是， 如果 值中存储 对象， 就算该对象的其他引用全都被移除 ， 也会阻止垃圾回收









#### 使用 Weak Map

> Weak Map 是 键值对的无序列表， 其中 键 必须是 非null 的对象， 值允许是任意类型， WeakMap  和  Map 非常相似， 都是用 set() 与 get() 方法类分别添加与提取数据

``` javascript
let map = new WeakMap(),
    element = document.querySelector('.element')

map.set(element, 'original')

let value = map.get(element)   // 'original'

// 移除元素
element.parentNode.removeChild(element)
element = null

// 该 Weak Map 在此处为空
```









#### Weak Map 的初始化

``` javascript
let key1 = {},
    key2 = {},
    map = new Map([ [key1, 5],[key2, 'hello'] ])   // key 必须是 非null 的对象

console.log(map.get(key1))   // 5
console.log(map.has(key1))   // true
```







#### Weak Map 的方法

> Weak Map 只有两个附加方法能用来与 键值对交互， has() 方法用于判断指定键是否存在于Map 中， 而 delete() 方法则 用于移除 一个特定 的键值对， 与 Weak Map 相同， 枚举Weak Map 也不可能， 因此 clear() 方法也不存在









#### 对象的私有数据

> 虽然 大多数开发者认为 weak map 的主要用途是 关联 数据 与 dom 元素， 但仍还有许多可能的用法， 比如创建 私有数据

``` javascript
// ES5 中 能够创建几乎真正私有的数据， 只要在创建对象时 使用类似下面的模式

var Person = (function() {
    
    var privateData = {},
        privateId = 0
    
    function Person(name) {
        Object.defineProperty(this, '_id', {value: privateId++})
        
        privateData[this._id] = {
            name
        }
    }
    
    Person.prototype.getName = function() {
        return privateData[this._id].name
    }
    
    return Person
}())
```

> 此例 用 IIFE 包裹了 Person 的定义， 其中含有两个私有变量，   privateData存储了所有实例的私有信息， 而 privateID 则被用于 为每个 实例产生一个唯一的 ID， 当 Person 构造器被调用时， 一个不可枚举， 不可配置， 不可写入的 _id 属性就被添加了

> 接下来 在 privateData 对象中 建立了与实例ID对应的一个入口， 其中存储着name 的值， 随后在 getName() 函数中， 就能使用this._id 作为 privateData 的键来提取该值， 由于 privateData 无法在 IIFE 外部被访问， 实际的数据就是安全的， 尽管 this._id 在 privateData对象上仍然是公开暴露的

> 此方式的最大的问题是 privateData 中的数据永不会消失， 因为在对象实例被销毁时没有任何方法可以获知改数据 ， privateData 对象就将永远包含多余的数据， 这个问题可以改用 Weak Map 来解决

``` javascript
let Person = (function() {
    let privateData = new WeakMap()
    
    function Person(name) {
        privateData.set(this, {name})
    }
    
     Person.prototype.getName = function() {
        return privateData.get(this).name
    }
}())
```







#### Weak Map 的用法与局限性

> 当需要在 Weak Map 与正规的 Map 中做出抉择时， 首要考虑的因素在于你是否只想使用对象类型的键， 如果是， 最好的选择是  Weak Map

> 另外一点是 Weak Map 只为他的内容提供了 很低的可见度， 因此不能用 forEach() 方法， size 属性， 或 clear()  来管理其中的项目