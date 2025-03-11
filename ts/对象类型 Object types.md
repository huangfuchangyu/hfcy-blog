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

虽然 typescript 同时支持 `string` 和 `number` 类型，但**数字索引的返回类型一定要是字符索引返回类型的子类型**，这是因为当一个数字进行索引的时候，JavaScript 实际上要把他转成一个字符串，这意味着使用数字100和字符串100是一样的

``` typescript
interface Animal {
  name: string
}

interface Dog extends Animal {
  breed: string
}

// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOK {
  [x: number]: Animal
  // 'number' index type 'Animal' is not assignable to 'string' index type 'Dog'.
  [x: string]: Dog
}
```

> 根据 TypeScript 的规则，所有的字符串索引签名必须返回一个与数字索引签名相同或更宽松的类型。换句话说，如果你有一个数字索引签名返回 Animal 类型，那么字符串索引签名也必须返回 Animal 类型或其子类型。

然而，如果一个索引签名是属性类型的联合，那各种类型的属性就可以接受了

``` typescript
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

当然，索引签名也支持 `readonly`

``` typescript
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
```



#### 接口继承（Extending Types）

有时我们需要一个比其他类型更具体的类型

``` typescript
interface BasicAddress {
  name?: string
  street: string
  city: string
}

interface AddressWithUnit extends BasicAddress {
  unit: string
}
```

对接口使用 `extends` 关键字允许我们有效的从其他声明过的类型中拷贝成员，并且添加其他成员, 接口也可以继承多个类型

``` typescript
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {}
 
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
}
```





#### 交叉类型 （Intersection Types）

交叉类型用于合并已经存在的对象类型， 定义交叉类型需要使用`&` 操作符

``` typescript
interface Colorful {
  color: string
}
interface Circle {
  radius: number
}

type ColorfulCircle = Colorful & Circle

function draw(circle: ColorfulCircle) {
  // ...
}

// okay
draw({ color: "blue", radius: 42 })
// error
draw({ color: "blue", radius: "42" })
```





#### 接口继承与交叉类型

两种方式在合并类型上看起来很相似，但实际上还是有很大的不同，最原则性的不同在于冲突的处理，这也是你决定选择哪种方式的主要原因

``` typescript
interface Colorful {
  color: string;
}

interface ColorfulSub extends Colorful {
  color: number
}

// Interface 'ColorfulSub' incorrectly extends interface 'Colorful'.
// Types of property 'color' are incompatible.
// Type 'number' is not assignable to type 'string'.
```

但是交叉类型不会报错，会取交集

``` typescript
interface Colorful {
  color: string;
}

type ColorfulSub = Colorful & {
  color: number
}
```

所以 color 的类型是 `never`, 因为 `string` 和 `number` 没有交集



#### 泛型对象类型（Generic Object Types）

``` typescript
interface Box<Type> {
  contents: Type
}

let box: Box<string>
```

这样的好处是我们可以通过泛型函数来避免使用函数重载

类型别名也可以使用泛型

``` typescript
type Box<Type> = {
  contents: Type
}
```

类型别名不同于接口，可以描述的不止是对象类型，所以也可以用类型别名写一些其他种类的泛型帮助类型

``` typescript
type OrNull<Type> = Type | null;
 
type OneOrMany<Type> = Type | Type[];
 
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
           
type OneOrManyOrNull<Type> = OneOrMany<Type> | null
 
type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
               
type OneOrManyOrNullStrings = OneOrMany<string> | null
```

