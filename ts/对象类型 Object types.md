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





#### Array 类型

当我们这样写类型 `number[]` 或者 `string[]` 的时候， 实际上是 `Array<number>` 和 `Array<string>` 的简写

`Array` 本身就是一个泛型

``` typescript
interface Array<Type> {
  length: number
  pop(): Type | undefined
  push(...items: Type[]): number
}
```

现代JavaScript也提供其他是泛型的数据结构，比如 `Map<K, V>`,  `Set<T>` 和 `Promise<T>`,  因为 `Map` `Set` `Promise` 的行为表现， 他们可以跟任何类型搭配使用



#### `ReadonlyArray` 

`ReadonlyArray` 是一个特殊类型，它可以描述数组不能被改变

``` typescript
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...but we can't mutate 'values'.
  values.push("hello!");
  // Property 'push' does not exist on type 'readonly string[]'.
}
```

`ReadonlyArray` 主要是来做意图声明，当我们看到一个函数返回 `ReadonlyArray` ,就是在告诉我们不能去更改其中的内容, 当我们看到一个函数支持传入 `ReadonlyArray` 这就告诉我们可以放心的传入数组到这个函数中，而不用担心改变数组内容，`ReadonlyArray` 并不是我们可以用的构造器函数

``` typescript
new ReadonlyArray("red", "green", "blue");
// 'ReadonlyArray' only refers to a type, but is being used as a value here.
```

然而我们可以直接把一个常规数组赋值给 `ReadonlyArray`

``` typescript
const roArray: ReadonlyArray<string> = ['red', 'green', 'blue']
```

typescript 也针对 `ReadonlyArray<Type>` 提供了更简短的写法 `readonlyType[]`

有一点要注意， `Arrays` 和 `ReadonlyArray` 并不能双向赋值

``` typescript
let x: readonly string[] = []
let y: string[] = []

x = y // ok
y = x //  The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```





#### 元祖类型 Tuple Types

元祖类型是另外一种 `Array` 类型，当你明确知道数组包含多少个元素，并且每个位置元素的类型都明确知道的时候，就适合使用元组类型

``` typescript
type StringNumberPair = [string, number]
```

在这个例子中，`StringNumberPair`  就是 `string` 和 `number` 的元组类型

如果要获取元素数量之外的元素，typescript 会提示错误

``` typescript
function doSomething(pair: [string, number]) {
  // ...
  const c = pair[2]
  // Tuple type '[string, number]' of length '2' has no element at index '2'.
}
```

也可以使用JavaScript 数组的解构语法解构元组

除了长度检查，简单的元组类型跟声明了 `length` 属性和具体的索引属性的 `Array` 是一样的

``` typescript
interface StringNumberPair {
  length: 2
  0: string
  1: number
  slice(start?: number, end?: number): Array<string | number>
}
```

在元组类型中，可以写一个可选属性，但是必须在最后面，而且也会影响 类型的`length` 

``` typescript
type Either2Or3 = [number, number, number?]
function setCoordinate(coord: Either2Or3) {
  const [x, y, z] = coord
  const z:number | undefined
  console.log(`Provided coordinates had ${coord.length} dimensions`);
  // (property) length: 2 | 3
}
```

Tuples 也可以使用剩余元素的语法，但必须是 array / tuple 类型

``` typescript
type StringNumberBooleans = [...boolean[], string, number]
```

有剩余元素的元组并不会设置 `length` 因为它只知道在不同位置上的一直元素信息

可选元素和剩余元素的存在，使得 typescript 可以在参数列表里使用元组

``` typescript
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}

// 基本等同于

function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```





#### `readonly` 元组类型

元组类型也是可以设置 `readonly ` 的

``` typescript
function doSomething(pair: readonly [string, number]) {
  // ...
  
  pair[0] = 'hello'
  // Cannot assign to '0' because it is a read-only property.
}
```

在大部分代码中， 元组只是被创建，使用完后也不会被修改，所以尽可能将元组设置为 `readonly` 是一个好习惯

如果给数组字面量 `const` 断言，也会被推断为 `readonly` 元祖类型

``` typescript
let point = [3, 4] as const;
 
function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}
 
distanceFromOrigin(point);

// Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
// The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'.


// 尽管  distanceFromOrigin  并没有更改传入的元素，但函数希望传入一个可变元组， 因为 point 的类型被推断为  readonly[3, 4], 他跟 [number, number] 并不兼容，所以 typescript 给了一个报错
```

