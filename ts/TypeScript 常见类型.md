## TypeScript 常见类型



#### 原始类型： `string`, `number`, `boolean`

JavaScript 有三个常用的原始类型： `string` `number` `boolean` ，每一种在 typescript 中都有对应的类型，使用 `typeof` 操作符 跟在 JavaScript 中的结果是一样的





#### 数组（Array）

声明一个类似于 `[1,2,3]`  的数组类型， 你需要使用语法 `number[]` , 也可以使用这种语法 `Array<number>` 

> 注意： [number] 和 number[] 表示不同的意思， 前者表示 元组





#### `any`

typescript 有一个特殊的类型 `any` ，当你不希望一个值导致类型检查错误的时候，就可以设置为 `any`

当一个值是 `any` 的时候，你可以获取他的任意属性（也会被转为 `any`  类型）

``` typescript
let obj: any = {a: 1}
obj.foo()
obj()
obj.bar = 100
obj = 'hello'
const n: number = obj
```

当你不想写一个长长类型的代码，仅仅想让 typescript 知道某段特定类型的代码是没有问题的， `any` 类型是很有用的





#### `noImplicitAny`

如果你没有指定一个类型，typescript 也不能从上下文中推断出他的类型，编译器就会默认为 `any` 类型

如果你总是想避免这种情况，可以开启编译项 `noImplicitAny` ，当被隐式的推断为 `any` 时， typescript 就会报错





####  变量上的类型注解

当你声明一个变量时，你可以选择性的添加一个类型注解

``` typescript
let myName: string = 'bob'
```

但这不是必须的， typescript 会基于初始值自动推断出类型





#### 函数

当你声明一个函数的时候，你可以在每个参数后面添加一个类型注解

``` typescript
function greet(name: string) {
  // ...
}
```

也可以添加返回值类型注解

``` typescript
function getNumber(): number {
  return 123
}
```

跟变量类型注解一样，不需要显式的添加返回值类型注解，typescript 会根据 `return` 语句自动推断出函数的返回值类型， 但一些代码库会显式的指定返回值类型，可能是需要编写文档，或者阻止意外修改，或者个人风格喜好







#### 匿名函数

匿名函数有一点不同于函数声明，当typescript 知道一个匿名函数将被怎样调用的时候，匿名函数的参数会被自动推断出指定类型， 这个过程被称为 **上下文推断** ， 从函数出现的上下文中推断出它应有的类型

``` typescript
const names  ['bob', 'jacky', 'tony']
names.foreach(function(s) {
  // 注意大小写
  console.log(s.toUppercase())
  // Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'
})
```





#### 对象类型

定义一个对象类型，只需要简单的列出他的属性和对应的类型

``` typescript
function printCoord(pt: {x: number, y: number}) {
  // ...
}

printCoord({x: 1, y: 2})
```

这里我们给参数添加了一个类型， `x` 和 `y`  两个都是 `number` 类型， 可以使用 `,` 或者 `;` 分开属性，最后一个属性分隔符加不加就行

每个属性如果不指定类型，就是 `any` 类型







#### 可选属性

对象类型可以指定一些甚至所有属性为可选的，只需要在属性名后添加一个 `?`

``` typescript
function printName(obj: {firstName: string, lastName?: string}) {
  // ...
}

// both ok
printName({firstName: 'bob', lastName: 'brown'})
printName({firstName: 'bob'})
```

在 typescript 中，当你获取一个可选属性时， 使用它前需要检查是否是 `undefined`





#### 联合类型

typescript 类型系统允许根据已经存在的类型构建新的类型

一个联合类型是由两个或者更多个类型组成，这其中的每个类型都是联合类型的**成员**

``` typescript
function printId(id: number | string) {
  // ...
}

// both ok
printId(10001)
printId('10001')
```

如果你有一个联合类型 `string | number`  你不能使用只存在 `string` 上的方法

``` typescript
function printId(id: number | string) {
  console.log(id.toUpperCase())
  // Property 'toUpperCase' does not exist on type 'string | number'.
  // Property 'toUpperCase' does not exist on type 'number'
}
```

解决方案是 使用代码收窄联合类型

``` typescript
function printId(id: number | string) {
  if (typeof id === 'string') {
    // ...
  } else {
    // ...
  }
}
```

再举个例子 `Array.isArray`

``` typescript
function welPeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // ...
  } else {
    // ...
  }
}
```

有时候，如果联合类型的每个成员都有一个属性，比如 数字和字符串都有 `slice` 方法，就可以直接使用而不用类型收窄

``` typescript
function getPreThree(x: number[] | string) {
  return x.slice(0, 3)
}
```

