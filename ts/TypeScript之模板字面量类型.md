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

模板字面量最有用的地方在于你可以基于一个类型内部的信息，定义一个洗的字符串

