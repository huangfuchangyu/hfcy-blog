## TypeScript 映射类型



有时候 一个类型需要基于另一个类型，但是又不想再拷贝一份，就可以使用映射类型

映射类型建立在索引签名的语法上，先回顾一下索引签名

``` typescript
type OnluBoolsAndHorses = {
  [key: string]: boolean | Horse
}

const conforms: OnluBoolsAndHorses = {
  del: true,
  rodney: false
}
```

映射类型，就是使用了 `PropertyKeys` 联合类型中的泛型，其中 `PropertyKeys` 多是通过 `keyof` 创建，然后循环遍历健名创建一个类型

``` typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean
}
```

`OptionsFlags` 会遍历 `Type` 所有的属性， 然后设置为布尔类型

``` typescript
type FeatureFlags = {
  darkMode: () => void
  newUsrProfile: () => void
}

type FeatureOptions = FeatureFlags<FeatureFlags>

// 结果为
type FeatureOptions = {
  darkMode: boolean
  newUsrProfile: boolean
}
```





#### 映射修饰符

使用映射类型时， 有两个额外的修饰符可能会用到，一个是 `readOnly` 用于设置属性只读， 一个是 `?`,  用于设置属性可选

可以通过前缀 `-` 或者 `+`  删除或者添加修饰符, 如果没有写前缀，相当于使用了 `+ ` 前缀

> 删除属性中的只读属性

``` typescript
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property]
}

type LockedAccount = {
  readonly id: string
  readonly name: string
}

type UnlockedAccount = CreateMutable<LockedAccount>

// 结果为：

// type LockedAccount = {
//   id: string
//   name: string
// }
```

> 删除属性中的可选属性

``` typescript
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property]
}

type MaybeUser = {
  id: string
  name?: string
  age?: number
}

type User = Concrete<MaybeUser>

// 结果为
// type User = {
//   id: string
//   name: string
//   age: number
// }
```







#### 通过 `as` 实现健名重新映射

在 TypeScript 4.1 及之后， 你可以在映射类型中使用 `as` 语句实现健名重新映射

``` typescript
type MappedTypeWithNewProperties<Type> = {
  [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

举个例子， 可以利用 模板字面量类型 基于之前的属性名创建一个新的属性名

``` typescript
type Getters<Type> = {
  [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]   
}

interface Preson {
	name: string
	age: number
}

type LazyPerson = Getters<Person>

// 结果为
type LazyPerson = {
	getName: () => string
	getAge: () => number
}
```

也可以利用条件类型返回一个 `never` 从而过滤掉某些属性

``` typescript
type RemoveKindField<Type> = {
  [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
}

interface Circle {
  kind: 'circle'
  radius: number
}

type KindlessCircle = RemoveKindField<Circle>

// 结果为
// type KindlessCircle = {
//   radius: number
// }
```

还可以遍历任何联合类型，不仅仅是 `string | number | symbol` 这种联合类型， 还可以是任何联合类型





#### 深入探索

映射类型也可以跟其他类型搭配使用， 举个例子 ， 这是一个使用条件类型的映射类型，根据对象是否有 `pii` 属性返回 `true` 或者 `false`

``` typescript
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends {pii: true} ? true : false
}

type DBFields = {
  id: {format: 'incrementing'}
  name: {type: string, pii: true}
}

type ObjectNeedingGDPRDeletion = ExtractPII<DBFields>

// 结果为
// type ObjectNeedingGDPRDeletion = {
//   id: false
//   name: true
// }
```

