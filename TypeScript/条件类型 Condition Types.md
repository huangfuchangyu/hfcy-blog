## 条件类型 Condition Types

有时候我们需要基于输入的值来决定输出的值，或者通过输入的类型来决定输出的类型。这种情况我们就需要使用 条件类型

``` typescript
interface Animal {
  live(): void
}

interface Dog extends Animal {
  voof(): void
}

type Example1 = Dog extends Animal ? number : string   // type Example1 = number
type Example2 = RegExp extends Animal ? number : string // type Example2 = string
```

规则如下 

> 类似JavaScript中的条件表达式：   condition ? trueExpression : falseExpression

``` typescript
Type1 extends Type2 ? TrueType : FalseType
```

当搭配泛型使用时，会展现出更大的作用

``` typescript
interface IdLabel {
  id: number
}

interface NameLabel {
  name: string
}

function createLabel(id: number): IdLabel
function createLabel(name: string): NameLabel
function createLabel(nameOrId: string | number): IdLabel | NameLabel
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw 'unimplemented'
}
```

这里使用了函数重载，这里的问题是：

* 如果一个库不得在一遍又一遍的去遍历API后作出相同选择，会变得很笨重
* 如果参数又新增了一种类型，那么重载的数量又会增加

这时，就可以使用条件类型简化掉函数重载

``` typescript
type NameOrId<T extends number | string> = T extends number ? IdLabel : NameLabel

function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw 'unimplemented'
}
```





#### 条件类型约束（Condition Type Constraints）

使用条件类型会为我们提供一些新的信息，正如使用 类型保护(type guards) 可以 收窄类型(narrowing) 为我们提供一个更加具体的类型，条件类型的 `true` 分支也会进一步约束泛型

``` typescript
type MessageOf<T> = T['message']
// type 'message' cannot be used to index tpe 'T'
```

TypeScript 报错是因为 `T` 不知道一个名为 `message` 的属性，我们可以这样约束 `T`

``` typescript
type MessageOf<T extends {message: unknow}> = T['message']

interface Email {
  message: string
}

type EmailMessageContents = MessageOf<Email>
// type EmailMessageContents = string
```

如果我们想要 `MessageOf` 可以传入任何类型，但是当传入的值没有 `message` 属性的时候，返回默认类型比如 `never`

``` typescript
type MessageOf<T> = T extends {message: unknow} ? T['message'] : never

interface Email {
  message: string
}

interface Dog {
  bark(): void
}

type EmailMessageContents = MessageOf<Email>
// type EmailMessageContents = string

type DogMessageCOntents = MessageOf<Dog>
// type DogMessageContents = never
```

在`true` 分支里，TypeScript 会知道 `T` 有一个 `message` 属性

``` typescript
// Flatten 类型，用于获取数组元素类型
// 当传入的不是数组，直接返回传入的类型
type Flatten<T> = T extends any[] ? T[number] : T // 这里使用了 《索引访问类型》里的 number索引，用于获取数组元素类型

type Str = Flatten<string[]>
// type Str = string

type Num = Flatten<number>
// type Num = number
```





#### 在条件类型里推断（Inferring With Condition Types）

条件类型提供了 `infer` 关键词， 可以从正在比较的类型中推断类型，然后在 `true` 分支里引用该推断的结果，使用 `infer` 从新写一下`Flatten`：

``` typescript
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type
```

`infer` 也可以用于写一些 类型帮助别名(helper type aliases), 下面的例子用于获取函数返回的类型：

``` typescript
type GetReturnType<Type> = Type extends (...args: never[]) => refer Return ? Return : never

type Num = GetReturnType<() => number>
// type Num = number

type Str = GetReturnType<(x: string) => string>
// type Str = string

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>
// type Bools = boolean[]
```

当从多重调用签名中推断类型的时候，比如函数重载,会按照最后的签名进行推断，因为一般这个签名是用来处理所有情况的说明

``` typescript
declare function stringOrNum(x: string): number
declare function stringOrNum(x: number): string
declare function stringOrNum(x: string | number): string | number

type T1 = ReturnType<typeof stringOrNum>
// type T1 = string | number
```



#### 分发条件类型（Distributive Conditional Types）

在泛型中使用条件类型的时候，如果传入一个联合类型，就会变成**分发的(distributive)** 

``` typescript
type ToArray<Type> = Type extends any ? Type[] : never

// 如果我们在 ToArray 传入一个联合类型， 这个条件类型会被用到每个联合类型的每个成员

type StrArrayOrNumArr = ToArray<string | number>
// type StrArrayOrNumArr = string[] | number[]
```



