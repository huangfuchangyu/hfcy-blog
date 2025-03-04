## 索引访问类型（indexed access type）

索引访问类型用于查找另一个类型上的特定类型

``` typescript
type Person = {age: number, name: string, alive: boolean}
type Age = Person['age']
// type Age = number
```

因为索引名本身是一个类型，所以我们可以使用联合、`keyof` 后者其他类型

``` typescript
type I1 = Person['age' | 'name']
// type I1 = string | number

type I2 = Person[keyof Person]
// type I2 = number | string | boolean

type AliveOrName = 'alive' | 'name'
type I3 = Person[AliveOrName]
// type I3 = string | boolean

// 不存在的属性会报错
type I4 = Person['alive']
// property 'alive' does not exist on type 'Person'
```



还可以使用`number` 来获取数组元素的类型， 结合`typeof` 可以方便的捕获数组字面量的元素类型

``` typescript
const MyArray = [
  {name: 'alice', age: 15},
  {name: 'bob', age: 20}
]

type Person = typeof MyArray[number]

// type Person = {
//  		name: string
//			age: number
// }

type Age = typeof MyArray[number]['age']
// type Age = number
```

作为索引的只能是类型, 不可以使用变量

``` typescript
const key = 'age'
type Age = Person[key]
// Type 'key' cannot be used as an index type.
// 'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?

// 可以使用类型别名
type key = 'age'
type Age = Person[key]
```

举个例子

> 有这样一个业务场景，一个H5页面要嵌入到不同APP里，比如 淘宝， 天猫，支付宝等，在不同的环境里，调用的API不同

``` typescript
const APP = ['TaoBao', 'Tmall', 'Alipay']
function getPhoto(app: string) {
  // ...
}

getPhoto('TaoBao')
```

但是这里 `getPhoto`入参约束为 `string` 类型，即使传入其他类型，也不会报错，这时可以使用字面量联合类型约束一下

``` typescript
const APP = ['TaoBao', 'Tmall', 'Alipay']
type app = 'TaoBao' | 'Tmall' | 'Alipay'

function getPhoto(app: app) {
  // ...
}

getPhoto('TaoBao')
```

这里写两遍类型有些冗余，可以继续优化

``` typescript
const APP = ['TaoBao', 'Tmall', 'Alipay'] as const
// as const 将数组变为 readonlu 的元祖类型
// const APP: readonly ['TaoBao', 'Tmall', 'Alipay'] 

type app = typeof APP[number]

function getPhoto(app: app) {
  // ...
}

getPhoto('TaoBao')
```

