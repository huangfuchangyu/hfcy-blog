## `keyof` 类型操作符

对一个对象使用`keyof` 操作符，会返回该对象属性明组成的一个字符串或者数字字面量的联合

``` typescript
type Point = {x: number, y: number}
type P = keyof Point
// type P = 'x' | 'y'
```

但是如果这个类型有一个 `string` 或者 `number` 类型的索引， `keyof` 会直接返回这些类型

``` typescript
type Arrayish = { [n: number]: unknow}
type A = typeof Arrayish
// type A = number

type Mapish = { [k: string]: boolean}
type M = keyof Mapish
// type M = string | number
```

注意这里， `M` 是 `string | number` ，因为 JavaScript 对象的属性名会被强制转换为一个字符串

> 这里我表示疑惑 0.0

> 文档到这里就结束了





#### 数字字面量联合类型

``` typescript
const NumbericObject = {
  [1]: 'num 1',
  [2]: 'num 2',
  [3]: 'num 3'
}

type result = keyof typeof NumbericObject
```

`typeof NumbericObject` 的结果为

``` typescript
{
  1: string,
  2: string,
  3: string
}
```

所以最终结果为

``` typescript
type result = 1 | 2 | 3
```





#### Symbol

typescript 也支持 `symbol` 类型的属性名

``` typescript
const sym1 = Symbol()
const sym2 = Symbol()
const sym3 = Symbol()

ocnst symbolToNumberMap = {
  [sym1]: 1,
  [sym2]: 2,
  [sym3]: 4,
}

type KS = keyof typeof symbolToNumberMap
// typeof sym1 | typeof sym2 | typeof sym3
```

