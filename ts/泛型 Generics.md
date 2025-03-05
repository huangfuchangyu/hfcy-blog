## 泛型 `Generics`

在比如 `C#` 和 `JAVA` 语言中，用来创建可复用的组件工具， 我们称之为泛型（`Generics`）, 利用泛型，我们可以创建一个支持众多类型的组件，折让用户可以使用自己的类型消费这些组件



#### 泛型初探

第一个泛型例子，一个恒等函数，就是返回任何传入的内容的函数

``` typescript
function identity(arg: number): number {
  return arg
}
```

或者使用 `any`

``` typescript
function identity(arg: any): any {
  return arg
}
```

这样实现会让我们丢失函数返回时的类型信息

所以我们需要一种可以捕获参数类型的方式，然后用它表示返回值的类型，这里我们使用一个 **类型变量(type variable)**, 一种用在类型而非值上的特殊变量

``` typescript
function identity<Type>(arg: type): type {
  return arg
}
```

我们写了一个泛型恒等函数之后，有两种方式调用它

``` typescript
// 第一种
let output = identity<string>('myString')

// 第二种
let output = identity('myString')
```

第二种方式更常见一些 ，这里我们使用了 **类型参数推断**，希望编译器能基于我们传入的参数自动推断和设置 `Type` 的值





#### 使用泛型类型变量

当你创建泛型函数时， 你会发现，编辑器会强制你在函数体内，正确的使用这些类型参数

``` typescript
function identity<Type>(arg: Type): Type {
  return arg
}
```

如果我们想打印 arg参数的长度

``` typescript
function identity<Type>(arg: Type): Type {
  console.log(arg.length)
  // property 'length' does not exist on type 'Type'
  return arg
}
```

我们也可以这样写，编辑器不会报错

``` typescript
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length)
  // Array has a .length, so no more error
  return arg
}
```



#### 泛型类型

泛型函数的形式就跟其他非泛型函数一样，需要列一个类型参数列表，有点像函数声明

``` typescript
function identity<Type>(arg: Type): Type {
  return arg
} 

let myIdentity: <Type>(arg: Type) => Type = identity

// 泛型类型参数可以使用不同的名字
let myIdentity: <Input>(arg: Input) => Input = identity

// 也可以以对象类型的调用签名的形式，书写这个泛型类型
let myIdentity: { <Type>(arg: Type): Type } = identity
```

这可以引导我们写出第一个泛型接口 ，让我们使用上个例子中的对象字面量，然后把它的代码移动到接口里

``` typescript
interface GenericIdentityFn {
  <Type>(arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
} 

let myIdentity: GenericIdentityFn = identity
```

有时候， 我们希望将泛型参数作为整个接口的参数，这可以让我们清楚的知道传入的是什么参数，而且接口里其他成员也可以看到

``` typescript
interface GenericIdentityFn<Type> {
  (arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
} 

let myIdentity: GenericIdentityFn<number> = identity
```



``` typescript
// PS: 
关于这两种写法的区别

interface GenericIdentityFn1 {
  <Type>(arg: Type): Type
}

interface GenericIdentityFn2<Type> {
  (arg: Type): Type
}

```

``` javascript
主要区别

泛型参数的作用域：
GenericIdentityFn1 的泛型参数 Type 是在函数调用时定义的，适用于每次调用。
GenericIdentityFn2 的泛型参数 Type 是在接口定义时指定的，适用于整个接口。

实现方式：
GenericIdentityFn1 允许在每次调用时使用不同的类型。
GenericIdentityFn2 在定义接口时就确定了类型，适用于特定类型的实现。

总结
如果您希望在每次调用时都能使用不同的类型，使用 GenericIdentityFn1。
如果您希望在接口定义时就确定类型，使用 GenericIdentityFn2。
```





#### 泛型类

泛型类的写法类似于泛型接口，在类名后面，使用尖括号 `<>` 包裹住类型参数列表

``` typescript
class GenericNumber<T> {
  zeroValue: T
  add: (x: T, y: T) => T
}

let myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0
```





#### 泛型约束

``` typescript
function identity<Type>(arg: Type): Type {
  console.log(arg.length)
  // property 'length' does not exist on type 'Type'
  return arg
}
```

对于这个例子，相比与兼容任何类型， 我们更愿意约束这个函数，让他只能使用带有 `.length` 属性的类型

为此，我们创建一个接口，用来描述约束

``` typescript
interface Lengthwise {
  length: number
}

function identity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length)
  // Now we know it has a .length property, so no more error
  return arg
}
```

现在这个泛型函数被约束了，它不再适用于所有类型

``` typescript
loggingIdentity(1)
// Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.

// 我们需要传入符合约束的值
loggingIdentity({ length: 1, value: 'xxx' })
```





#### 在泛型约束中使用类型参数

你可以声明一个类型参数，这个类型参数被其他类型参数约束

比如我们希望获取一个对象给定属性名的值，因此，我们要确保不会获取到对象上不存在的属性，所以我们在两个类型之间建立一个约束

``` typescript
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}

let x = {name: 'bob', age: 20}

getProperty(x, 'name') // ok
getProperty(x, 'sub') // Argument of type '"sub"' is not assignable to parameter of type '"name" | "age"'
```





#### 在泛型中使用类类型(Using Class Type in Generics)

在 TypeScript 中， 当使用工厂模式创建实例的时候，有必要通过他们的构造函数推断出类的类型

``` typescript
function create<Type>(c: {new (): Type}): Type {
  return new c()
}
```

``` typescript
代码解析：

1. 泛型参数：

Type 是一个泛型参数，表示函数可以接受任何类型。使用泛型的好处是可以在函数调用时指定具体的类型，从而提高代码的灵活性和重用性。

2. 函数参数：

c: { new (): Type }：这个参数 c 是一个构造函数类型。它要求 c 是一个可以被 new 关键字调用的构造函数，并且该构造函数在被调用时返回一个 Type 类型的实例。
{ new (): Type } 的意思是：c 是一个构造函数，没有参数（()），并且返回类型为 Type。

3. 函数体：

return new c();：这行代码使用 new 关键字调用构造函数 c，并返回一个新的实例。这个实例的类型是 Type。
```

下面是一个更复杂的例子， 使用原型属性推断和约束，构造函数和类实例的关系

``` typescript
class BeeKeeper {
  hasMask: boolean = true;
}
 
class ZooKeeper {
  nametag: string = "Mikle";
}
 
class Animal {
  numLegs: number = 4;
}
 
class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}
 
class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}
 
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
 
createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

