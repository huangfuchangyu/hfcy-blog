## JSON 

json  是一种轻量级的数据格式， 可以简化表示复杂数据结构的工作量， json 使用 JavaScript 的子集表示 对象， 数组， 字符串， 数值， 布尔， 和 null

即使 xml 也能表示同样复杂的数据结果， 但是 json 没有那么繁琐 ， 而且在 JavaScript 中使用更便利

> 兼容情况：  IE8+  Firefox3.5+ Safari4+ Opera10.5 Chrome



**语法**

json 语法可以表示一下三种类型的值：

* 简单值
* 对象
* 数组

json 不支持 变量 ， 函数， 对象实例， 他就是一种表示结构化数据的格式



JSON 之所以流行， 拥有与 JavaScript 类似的语法并不是全部原因， 更重要的是， JSON 数据解构解析为有用的 JavaScript 对象 ， 与 xml 数据结构要解析成DOM 文档 而且 从提取数据极为麻烦相比， JSON 可以解析为 JavaScript 对象的优势极其明显

``` java
// eg:  
books[2].title

// xml: 
doc.getElementByTagName('book')[2].getArrribute('title')
```





**JSON.stringify()**

``` javascript
// 1. 过滤结果
var book = {
    "title": 'Professoinal Javascript',
    "auther": ['Zakas'],
    "year": 2011
}

var textJson = JSON.stringify(book, ['title', 'year'])
// 因此 在返回的结果中， 就只会包含这两个属性
//var book = {
//    "title": 'Professoinal Javascript',
//    "year": 2011
//}

// 2. 参数为函数的情况
var textJson = JSON.stringify(book, function(key, value) {
    switch(key){
        case 'title': 
            return 'hha'
        case 'year':
            return 2020
            
        default: 
          return value  
    }
})

3. 字符串缩进
// json.stringify 方法的第三个参数用于控制结果中的缩进和空白符， 如果这个参数是一个值， 那他表示的是每个级别缩进的空格数， 例如 要缩进四个空格 ， 可以这样写
var textJson = JSON.stringify(book,null, 4)
// 缩进的最大空格数为10 ， 大于 10  按10 处理
var textJson = JSON.stringify(book,null, '--')
// 同样  缩进字符串长度最大也是10 ， 长度超过10， 只显示 前10 位


```

 JSON.stringify() 将值转换为相应的JSON格式：

- 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
- 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
- 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
- `undefined`、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 `null`（出现在数组中时）。函数、undefined 被单独转换时，会返回 undefined，如`JSON.stringify(function(){})` or `JSON.stringify(undefined)`.
- 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
- 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 `replacer` 参数中强制指定包含了它们。
- Date 日期调用了 toJSON() 将其转换为了 string 字符串（同Date.toISOString()），因此会被当做字符串处理。
- NaN 和 Infinity 格式的数值及 null 都会被当做 null。
- 其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。



**toJSON() 方法**

有时， json.stringify() 还不能满足对某些对象进行自定义序列化的需求， 在这些情况下， 可以通过 toJSON 方法， 返回其自身的数据格式， 原生Date对象就有一个 toJSON ， 能够将 JavaScript 的 Date 对象自动转化为 ISO8601 日期字符串

``` javascript
var jsonStr = {"aaa": 123, toJSON: function() {return 123}}
console.log(JSON.stringify(jsonStr))
// 123
```

所以 序列化对象的顺序为： 

1. 如果存在 toJSON 方法， 而且能够通过他取得有效的值， 则调用该方法， 否则， 按照默认顺序执行序列化
2. 如果提供了第二个参数， 应用这个函数过滤器， 传入过滤器的值 是 第一步返回的值
3. 对第二步返回的每个值进行响应的序列化
4. 如果提供第三个参数， 执行响应的格式化





**JSON.parse()**

JSON.parse() 方法也可以接收另一个参数， 该参数是一个函数， 函数接受两个参数， 用法同 JSON.stringify 相同

``` javascript
var bookCopy = JSON.parse(jsonText, function(key, value) {
    if(key === 'releaseDate') {
    	return new Date(value)   
    }
})

// 如果返回的值是 undefined 则表示要删除相应的键 
```

