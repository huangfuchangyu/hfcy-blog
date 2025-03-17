模板字面量类型以字符串字面量类型为基础，可以通过联合类型扩展成多个字符串

它们跟 JavaScript 的模板字符串是相同的语法，但是只能用在类型操作中，当使用模板字面量类型时，他会替换模板中的变量，返回一个新的字符串字面量

``` typescript
type World = 'world'

type Greet = `hello ${World}` // type Greeting = 'hello world'
```

当模板中的变量是一个联合类型时，每一个可能的字符串字面量都会被表示

``` typescript
type EmailLocaleIDs = 'welcome_email' | 'email_heading'
type FooterLocaleIDs = 'footer_title' | 'footer_sendoff'

type AllLocaleIds = `${EmailLocaleIDs | FooterLocaleIDs}_id`
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

如果模板字面量里的多个变量都是联合类型，结果会交叉相乘，比如下面的例子就有 2 * 2 * 3 一共 12 种结果

``` typescript
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`
type Lang = 'en' | 'js' | 'pt'

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`
// type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"

```



#### 类型中的字符串联合类型

模板字面量最有用的地方在于你可以基于一个类型内部的信息，定义一个新的字符串

有这样一个函数 `makeWatchedObject` 它会给传入的对象添加了一个  `on` 方法，在 JavaScript 中，它的调用看起来是这样的  `makeWatchedObject(baseObject)`  我们假设这个传入的对象为

``` typescript
const passedObject = {
  firstName: 'bob',
  lastName: 'james',
  age: 20
}
```

这个 `on` 方法会被添加到这个传入对象上，改方法添加了两个参数 `eventName`  string 类型， 和  `callBack` function 类型

> 伪代码

``` typescript
const result = makeWatchedObject(baseObject)
result.on(eventName, callBack)
```

我们希望 `eventName` 是这总形式： `attributeInThePassedObject + "Changed" `   , 举个例子 `passedObject` 有一个属性 `firstName`  ，对应产生的 `eventName` 为 `firstNameChanged` , 同理， `lastName` 对应的是 `lastNameChanged` ， `age` 对应的是 `ageChanged`

当这个 `callBack` 函数被调用的时候

* 应该被传入与  `attirbuteInThePassedObject` 相同类型的值，比如 `passedObject` 中，firstName 的值类型为 `string` ，对应 `firstNameChanged` 事件的回调函数，则接受传入一个 `string` 类型的值
* 返回值类型为 `void` 类型

`on()` 方法的签名最开始是这样的 `on(eventName: string, callBack: (newValue: any) => void)`

使用这样的签名，是不是就能实现上面所说的这些约束，这个时候就可以使用模板字面量

``` typescript
const person = makeWatchedObject({
  firstName: 'bob',
  lastName: 'jack',
  age: 20
})

person.on(
	'firstNameChanged',
  newValue => {
    console.log(`firstName was changed to ${newValue}`)
  }
)
```

在这个例子里 ， `on` 方法添加的事件名称为 `firstNameChanged` 而不仅仅是 `firstName` ，而回调函数传入的值 `newValue` 我们希望约束为 `string` 类型

在这个例子里， 我们希望传入的事件名的类型，是对象属性名的联合，只是每个联合成员都还在最后拼接一个 `Changed` 字符，在 JavaScript 中，我们可以做这样一个计算
``` typescript
Object.keys(passedObject).map(x => ${x}Changed)
```

模板字面量提供了一个相似的字符串操作

``` typescript
type PropEventSource<Type> = {
  on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void
}

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>
```

在这个例子中，模板字面量里我们写的是 `string & keyof Type` , 我们可不可以只写成 `keyof Type` ？ 这样写会报错的

``` typescript
type PropEventSource<Type> = {
  on(eventName: `${keyof Type}Changed`, callback: (newValue: any) => void): void
}

// Type 'keyof Type' is not assignable to type 'string | number | bigint | boolean | null | undefined'.
// Type 'string | number | symbol' is not assignable to type 'string | number | bigint | boolean | null | undefined'.
// ...

```

原因就是 `keyof` 操作符会返回 `string | number | symbol` 类型 ，但是模板字面量的变量要求类型是 `string | number | bigint | boolean | null | undefined`   比较一下 多了个 `symbol` 类型，所以可以这样写

``` typescript
type PropEventSource<Type> = {
  on(eventName: `${Exclude<keyof Type, symbol>}Changed`, callback: (newValue: any) => void): void
}
```

或者这样写

``` typescript
type PropEventSource<Type> = {
  on(eventName: `${Extract<keyof Type, string>}Changed`, callback: (newValue: any) => void): void
}
```

这样，当我们使用错误的事件名时， TS 会给出报错



#### 模板字面量的推断

现在来实现第二点，回调函数传入的值的类型与对应的属性值的类型相同，现在只是简单的对 `callback` 的参数类型使用 `any` 类型，实现这个约束的关键在于使用泛型函数

1. 捕获泛型函数的第一个参数的字面量，生成一个字面量类型
2. 该字面量类型可以被对象属性构成的联合约束
3. 对象属性的类型可以通过索引访问获取
4. 应用此类型，确保回调函数的参数类型与对象属性的类型是同一个类型

``` typescript
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void ): void;
};
 
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
 
person.on("firstNameChanged", newName => {                             
														  // (parameter) newName: string
    console.log(`new name is ${newName.toUpperCase()}`);
});
 
person.on("ageChanged", newAge => {
                        // (parameter) newAge: number
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

这里我们把 `on` 改成了一个泛型函数

当一个用户调用的时候传入 `firstNameChanged` ， TypeScript 会尝试推断 `Key` 的正确类型， 他会匹配 `key` 和 `Changed` 前的字符串, 然后推断出字符串`firstName` ， 然后再获取原始对象 `firstName ` 属性的类型，在这个例子中， 就是 `string` 类型



#### 内置字符串操作类型

TypeScript 的一些类型可以用于字符操作，这些类型处于性能的考虑被内置在编辑器中，你不能在 `.d.ts` 文件里找到他们



###### Uppercase<String Type>

把每个字符转为大写形式

``` typescript
type Greeting = 'Hello World'
type ShoutyGreeting = Uppercase<Greeting> // type ShoutyGreeting = "HELLO WORLD"

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<str>}`
type MainID = ASCIICacheKey<"my_app"> // type MainID = "ID-MY_APP"
```





###### Lowercase<String Type>

把每个字符转为小写形式 ，同 `Uppercase`





###### Capitalize<String Type>

把字符串的第一个字符转为大写形式





###### Uncapitalize<String Type>

把字符串的第一个字符转为小写形式





###### 字符操作类型的技术细节

从 TypeScript 4.1 起， 这些内置函数会直接使用 JavaScript 字符串运行时函数，而不是本地化识别

``` typescript
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}

```

