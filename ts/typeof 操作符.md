## `typeof` 操作符

JavaScript 本身就有 `typeof` 操作符

``` typescript
console.log(typeof 'hello world')
```

TypeScript 添加的 `typeof` 方法可以在类型上下文 `type context` 中使用， 用于获取一个变量或者属性的类型

``` typescript
let s = 'hello'
let n: typeof s
// let n: string
```

如果仅仅用来判断基本类型，自然没什么大用，和其他类型操作符搭配使用才能发挥作用

比如搭配 TypeScript 内置的 `ReturnType<T>` , 它的作用是接受一个函数类型，返回这个函数的返回值类型

``` typescript
type Predicate = (x: unknow) => boolean
type K = ReturnType<Predicate>
// type K = boolean
```

如果直接对一个函数名使用 `ReturnType` 会报错

``` typescript
function f() { return {x: 10, y: 3} }
type P = ReturnType<f>
// 'f' refers to a value, but is being used as a type here. Did you mean 'typeof f'?

// 正确写法为
type P = ReturnType<typeof f>
```





#### 限制

TypeScript 有意的限制了可以使用 `typeof` 表达式的种类，只能对标识符，比如变量名或者他们的属性使用 `typeof` 才是合法的，比如： 

``` typescript
// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?")
// ',' expected.

// 看似可以正常运行，实际并不会
// 因为 typeof 只能对标识符和属性使用

// 正确的写法为：
ReturnType<typeof msgbox>
```

> 官方文档感觉就写了一半， 到这里就没了



#### 对象使用 `typeof` 

``` typescript
const person = {
  name: 'bob',
  age: 12
}

type Kevin = typeof person
// 结果为
type Kevin = {
  name: string
  age: number
}
```





#### 函数使用`typeof`

``` typescript
function identity<Type>(arg: Type): Type {
  return arg
}

type result = typeof identity
// type result = <Type>(arg: Type) => Type
```





#### 对 `enum` 使用 `typeof`

在 TypeScript 中 ， enum 是一种新的数据类型，在具体运行的时候，他会被编译成对象

``` typescript
enum UserResponse {
  No = 0,
  Yes = 1,
}
```

对应编译后的JavaScript 代码为

``` typescript
var UserResponse;
(function (UserResponse) {
    UserResponse[UserResponse["No"] = 0] = "No";
    UserResponse[UserResponse["Yes"] = 1] = "Yes";
})(UserResponse || (UserResponse = {}));
```

控制台打印的结果为

``` typescript
console.log(UserResponse)
// [LOG]: {
//   "0": "No",
//   "1": "Yes",
//   "No": 0,
//   "Yes": 1
// } 
```

对 `UserResponse` 使用 `typeof` 的结果为

``` typescript
```

对一个 enum 使用 `typeof` 一版没什么用， 通常会搭配 `keyof` 使用

``` typescript
type result = keyof typeof UserResponse
// type result = 'No' | 'Yes'
```

