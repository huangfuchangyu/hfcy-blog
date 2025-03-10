## 对象类型 Object types

对象类型可以是匿名的

``` typescript
function greet(person: {name: string, age: number}) {
  // ...
}
```

也可以使用接口进行定义

``` typescript
interface Person {
  name: string
  age: number
}

function greet(person: Person) {
  // ...
}
```

或者使用类型别名

``` typescript
type Person = {
	name: string
  age: number
}

function greet(person: Person) {
  // ...
}
```



#### 可选属性

在属性名后面加一个 `?` 标记这个属性是可选的

``` typescript
interface PaintOptions {
  shape: Shape
  xPos?: number
  yPos?: number
}
```



#### `readonly`

被标记为 `readonly` 的属性，不会改变任何运行时的行为，在类型检查的时候，一个标记为 `readonly` 的属性是不能被写入的

``` typescript
interface SomeType {
  readonly prop: string
}

function doSomething(obj: SomeType) {
  console.log(obj.prop) // ok
  obj.prop = 'hello' // Cannot assign to 'prop' because it is a read-only property.
}
```

使用 `readonly` 并不意味着一个值就完全不变的，它仅仅表明属性本身是不能被重新写入的

``` typescript
interface Home {
  readonly resident: { name: string, age: number}
}

function visitForBirth(home: Home) {
  home.resident.age++ // ok
  home.resident = {} // // Cannot assign to 'resident' because it is a read-only property.
}
```

typescript 在检查两个类型是否兼容的时候，并不会检查两个类型的属性是否是`readonly` 这意味着，`readonly` 是可以通过别名修改的

``` typescript
interface Person {
  name: string
  age: number
}

interface ReadonlyPerson {
  readonly name: string
  readonly age: number
}

let writablePerson: Person = {
  name: 'mike'
  age: 22
}

// ok
let readonlyPerson: ReadonlyPerson = writablePerson

console.log(readonlyPerson.age) // 22
writablePerson.age++
console.log(readonlyPerson.age) // 23
```



#### 索引签名

有时候，你不可能知道一个类型里所有属性的名字，但是你知道这些值的特征

``` typescript
interface StringArray {
  [index: number]: string
}
```

**一个索引签名的属性类型必须是`string` 或者 `number`**

虽然 typescript 同时支持 `string` 和 `number` 类型，但数字索引的返回类型一定要是字符索引返回类型的子类型，这是因为当一个数字进行索引的时候，JavaScript 实际上要把他转成一个字符串，这意味着使用数字100和字符串100是一样的

``` typescript
interface Animal {
  name: string
}

interface Dog extends Animal {
  breed: string
}
```

