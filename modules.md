## Modules

> js “共享一切” 的代码加载方式是该语言混乱且容易出错的方面之一， 其他语言使用package 之类的概念来定义代码的作用域， 然而在 ES6 之前， 一个应用的每个js 文件所定义的所有内容全都由全局作用域共享， 当应用变得更加复杂， 命名冲突， 安全等问题就会发生， ES6 的设计目的之一就是要解决作用域的问题

modules 是使用不同方式加载的 js 文件（与 js 原先的脚本加载方式相对），这种不同的模式与 script 有不同的语义：

* 模块代码自动运行在严格模式下，并且没有任何代码可以退出严格模式
* 在模块的顶级作用域创建的变量， 不会被自动添加到共享的全局作用域， 他们只会在模块顶级作用域内不存在
* 模块顶级作用域的this 值为 undefined 
* 模块不允许在代码中使用 HTML 风格的注释（这是js 来自于早期浏览器的历史遗留问题）
* 对于需要让模块外部代码访问的内容， 模块必须导出他们
* 允许模块从其他模块导入绑定



#### 基本的导出

可以使用 export 关键字将已发布代码部分公开给其他模块， 最简单的方法就是将 export 放置在任意的 变量  函数  或类声明之前

``` javascript
// 导出数据
export var color = 'red'
export let name = 'bob'
export const magicNumber = 7

// 导出函数
export function sum() {}

// 导出类
export class Rectangle {
    constructor() {}
}

// 次函数为模块私有
function subtract() {}

// 定义一个函数
function multiply() {}

// 稍后将其导出
export {multiply}
```

> 1. 除了 export 关键字之外， 每个声明都与正常形式完全一样
> 2.  每个导出的函数或类都有名称， 因为导出的函数与类声明都必须要有名称
> 3. 不能使用匿名函数导出 除非使用 default 关键字
> 4. 不仅可以导出声明， 还可以导出引用
> 5. 未导出的函数在模块外部不可以被访问， 因为灭有被显示的导出， 会保持模块的私有







#### 基本的导入

import 语句有两个部分， 一是需要导入的标识符， 而是需要导入的标识符的模块来源

``` javascript
import {identifier1, identifier2} from './examplee.js'
```

> 导入绑定的列表看起来与对象解构相似， 其实并无关系

当模块导入一个绑定是， 该绑定表现得就像使用了const 的定义， 意味着不能在定义一个同名变量（也包括导入另一个同名绑定）， 也不能在 import 语句之前使用此标识符（也是受到暂时性死区的限制），更不能修改他的值









#### 完全导入一个模块

还有一种特殊的情况， 允许将整个模块当做单一对象导入， 该对象的所有导出都会作为对象的属性存在

``` javascript
// 完全导入
import * as example from './example.js'

console.log(example.sum)
```

无论对同一个模块使用多少次import 语句 ， 该模块都只会被执行一次， 已被实例化的模块会被存在内存中， 并随时被其他import 所引用

``` javascript
import {sum} from './example.js'
import {multiply} from './example.js'
```









#### 模块语法的限制

export 与 import 都有一个重要的限制， 必须被用在其他语句或表达式的外部

``` javascript
if(flag){
    export flag    // 语法错误
} 
```

类似的  不能在一个语句内部使用 import 

``` javascript
function tryimport() {
    import flag from './example.js'
}
```





#### 导入绑定的一个微妙怪异点