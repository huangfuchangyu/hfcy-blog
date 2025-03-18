## More On Functions

本节介绍的是如何书写描述函数的类型

#### 函数类型表达式

``` typescript
function greeter(fn: (a: string) => void) {
  fn('hello word')
}

function printToConsole(s: string) {
  console.log(s)
}

greeter(printToConsole)
```

也可以使用类型别名定义一个函数类型

``` typescript
type GreetFunction = (a: string) => void

function greeter(fn: GreetFunction) {
  fn('hello word')
}
```





#### 调用签名 Call Signaures

在JavaScript中，函数除了可以被调用，也是可以有属性值的，如果想描述一个带有属性值的函数，可以在对象类型中使用调用签名

``` typescript
type DescFunc = {
  description: string
  (someArg: number): boolean
}

function doSomething(fn: DescFunc) {
  console.log(fn.description)
  console.log(description(666))
}
```







#### 构造签名 Consturct Signatures

JavaScript函数也可以使用 `new` 操作符调用， 当被调用的时候， TypeScript 会被认为这是一个构造函数, 因为他们会产生一个新对象， 你可以写一个构造签名，方法是在调用签名前面加一个 `new` 关键字

``` typescript
type SomeConstructor = {
  new (s: string): SomeObject
}

function fn(ctor: SomeConstructor) {
  return new ctor('hello')
}
```

一些对象 ， 比如 `Date` 对象， 可以直接调用，也可以使用 `new` 操作符调用, 你可以将调用签名和构造签名合并在一起

``` typescript
interface CallOrConstruct {
  new (s: string): Date
  (n?: number): number
}
```







#### 泛型函数 Generic Functions

我们经常需要写这种函数，即函数的输出类型依赖函数的输入类型, 或者两个类型以某种形式相互关联，让我们写这样一个函数，它返回数组的第一个元素

``` typescript
function firstElement(arr: any[]) {
  return arr[0]
}
```

此时函数的返回值类型是  `any` ， 如果能返回第一个元素的具体类型就更好了

``` typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0]
}
```

现在，我们在函数的输入和输出之间创建了一个关联，现在 当我们调用它，一个更具体的类型就会被判断出来

``` typescript
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```





#### 推断 Inference

在上面的例子中，我们没有明确指定 `Type` 类型，类型是被 自动推断出来的

我们也可以使用多个类型参数

``` typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func)
}

// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n))
```

在这里， TypeScript 既可以推断出 `Input` 的类型， 又可以推断出 `Output` 的类型





#### 约束 Constraints

有时，我们想对参数进行限制，比如要求参数都包含一些固定字段

让我们写一个函数，返回入参中更长的那个，我们需要保证每个参数中都有 `length` 属性

``` typescript
function longest<Type extends {length: number}>(a: Type, b: Type) {
  return a.length > b.length ? a: b
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
// Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.

```

TypeScript 会推断出 `longest` 的返回类型，所以返回值的类型推断在泛型函数里也是适用的





#### 泛型约束实战

这是一个泛型约束常见的错误

``` typescript
function minimumLength<Type extends {length: number}>(obj: Type, minimum: number): Type {
  return obj.length >= minimum ? obj : {length: minimum}
}
```

这个函数看起来没问题，`Type` 被 `{length: number}` 约束，函数返回 `Type` 或者一个符合约束的值

而 这其中的问题就是函数应返回与传入参数想同类型的对象，而不仅仅是符合约束条件

可以写这样一个反例

``` typescript
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```





#### 声明类型参数

TypeScript 通常会自动推断出传入的类型参数，但也并不能总是推断出，举个例子，有这样一个合并两个数组的函数

``` typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2)
}
```

如果你像下面这样调用就会出现错误

``` typescript
const arr = combine([1,2,3], ['hello'])
// Type 'string' is not assignable to type 'number'.

// 如果执意这么做， 可以调用时手动指定 Type
const arr = combine<string | number>([1,2,3], ['hello'])
```



