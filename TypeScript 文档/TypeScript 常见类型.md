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





#### 类型别名 `type`



#### 接口 `interface`





#### `type` 和 `interface` 的不同

大多数情况，两者可以任意使用，两者最关键的差别在于 类型别名本身无法添加新的属性，而接口是可以扩展的



**Interface 通过 extends 扩展类型**

``` typescript

interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}
```



**Type 通过 &  扩展类型**

``` typescript
type Animal = {
  name: string
}

type Bear = Animal & {
  honey: boolean
}
```



**Interface 可以对一个已经存在的接口添加新的字段**

``` typescript
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}
```



**Type 创建后不能被改变**

``` typescript
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

// Error: Duplicate identifier 'Window'
```



* 在 typescript 4.2以前， `type` 的名字可能会出现在报错信息中，有时会替代等价的匿名类型（也许并不是期望的），`interface` 的名字则会始终出现在错误信息中
* `type` 也许不会实现 声明合并 但是接口可以
* `interface` 可能只会被用于 声明对象的形状， 不能重命名原始类型
* `interface` 通过名字使用的时候， 他的名字总是会出现在错误信息中, 如果直接使用，则会出现原始结构 





####  类型断言

有时候 ，你知道一个值的类型，但是 typescript 不知道

比如， 如果你使用 `document.getElementById`,  `typescript`  仅仅知道它会返回一个 `HTMLElement` ， 但是你却知道，你要获取的是一个 `HTMLCanvasElement`, 这时，你可以使用断言将其指定为更具体的类型

``` typescript
const myCanvas = document.getElementById('my_canvas') as HTMLCanvasElement
```

就像类型注释，断言也会被编辑器移除，不会影响任何运行时行为

也可以使用尖括号语法，但是不能在 `.jsx` 文件内使用

``` typescript
const myCanvas = <HTMLCanvasElement>document.getElementById('my_canvas')
```

typescript 仅仅允许类型断言转换为一个更加具体或更加不具体的类型，这个规则可以阻止一些不可能的强制类型转换

``` typescript
const x = 'hello' as number
```

有时候，这条规则显得非常保守，阻止了原本有效的类型转换，你可以使用双重断言，先断言为 `any` 或者 `unknown` 然后再断言为期望的类型

``` typescript
const a = (expr as any) as T
```





#### 字面量类型

字面量类型本身并没有什么太大用

``` typescript
let x: "hello" = "hello"

x = "hello" // OK
x = "hollday" // Type "holiday" is not assignable to type "hello"
```

如果结合联合类型，就显得有用多了，比如，当函数只能传入一些固定的字符串时：

``` typescript
function printText(s: string, alignment: 'left' | 'right' | 'center') {}

printText('hello', 'left') // ok
printText('hello', 'bottom') // Argument of type '"bottom"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

数字类型字面量也同理

也可以跟非字面量的类型联合

``` typescript
interface Options {
  width: number
}

function configure(x: Options | 'auto') {}

configure({width: 100})
configure('auto')
configure('automatic') //Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

还有一种字面量类型，布尔字面量，因为只有 `true` , `false` 两种字面量类型， 类型 `boolean` 实际上就是联合类型 `true | false` 的别名





#### 字面量推断

当你初始化变量为一个对象的时候，TypeScript 会假设这个对象的属性的值未来会被修改

``` typescript
const obj = {
  counter: 0
}
obj.counter = 1

// obj.counter 必须是 string 类型，但不要求一定是 0
```

也同样应用于字符串

``` typescript
declare function handleRequest(url: string, method: "GET" | "POST"): void

const req = {url: 'baidu.com', method: 'GET'}
handleRequest(req.url, req.method)
// Argument of type 'string' is not assignable to parament of type 'GET' | 'POST'
```

有两种方式可以解决

1. 添加类型断言改变推断结果

   ``` typescript
   const req = {url: 'baidu.com', method: 'GET' as 'GET'}
   // 或
   handleRequest(req.url, req.method as 'GET')
   ```

2. 也可以使用 `as const` 把整个对象转为一个类型字面量

      ``` typescript
      const req = {url: 'baidu.com', method: 'GET'} as const
      handleRequest(req.url, req.method)
      ```

`as const` 效果跟 `const` 类似，但是对类型系统而言， 他可以确保所有的属性都被赋予一个字面量类型，而不是一个更通用的类型比如 `string` 或者 `number`



#### `strictNullChecks`

`strictNullChecks`  用于控制一个值可能是 `null` 或 `undefined` 你需要在使用他的方法或属性之前，使用类型收窄进行判断





#### 非空断言操作符（后缀！）

在任意表达式后面加上 `!` ，代表它的值不可能是 `null` 和 `undefined`  ， 这是一个有效断言

``` typescript
function liveDangerously(x?: number | null) {
  // no error
  console.log(x!.toFixed())
}
```





#### 不常见的原始类型



**bigint**



**symbol**
