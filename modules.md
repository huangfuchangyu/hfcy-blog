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

ES6 的import 语句为变量， 函数 与类 创建了只读绑定， 而不像普通变量那样简单引用了原始绑定， 尽管导入绑定的模块无法修改绑定的值， 但负责导出的模块却能做到这一点

``` javascript
export var name = 'bob'
export function setName(newName) {
    name = newName
}
```

当导入了这两个绑定后， setName 还可以改变name 的值

``` javascript
import {name, setName} from './example.js'

console.log(name)   // 'bob'
setName('Greg')    
console.log(name)   // ''Greg

name = 'Kyire'     // error
```







#### 重命名导出与导入

``` javascript
// 重命名导出
function sum() {}

export {sum as add}


// 导入 
import {add} from './example.js'
```

``` javascript
// 重命名导入
import {add as sum} from './example.js'
```









#### 导出默认值

模块语法为从模块中导出或导入默认值进行了优化， 模块的默认值使用 default 关键字所指定的单个变量，函数或类， 而在每个模块中只能设置一个默认导出，将default 关键字用于多个导出会导致语法错误

``` javascript
export default function sum() {}

// 或 

function sum（） {}
export default sum

// 或
export {sum as default}
```







#### 导入默认值

``` javascript
// 导出
export let color = 'red'
export default function sum() {}

// 导入
import sum , {color} from './example.js'

// 如果导出默认值， 也可以使用重命名语法对默认值进行导入
import {default as sum, color } from './example.js'
```







#### 绑定的再导出

有时想将导入的模块在重新导出

``` javascript
import {sum} from './example.js'
export{sum}

// 还可以使用单个语句完成相同的任务
export {sum} from './example.js'

// 使用别名
export {sum as totle} from './example.js'

// 如果想将来自于另一个 模块的所有值完全导出 可以使用 * 
// 假设example.js 具有一个默认导出， 当你使用这种语时
// 就无法为当前模块在定义一个默认导出
export * from './example.js'

```









#### 加载模块

尽管ES6定义了模块的语法，但并未定义如何加载他们， 这是规范复杂性的一部分， 这种复杂性对于实现环境来说是无法预知的， ES6 未选择给所有的js 环境努力创建一个有效的单一规范， 而只规定了语法， 并制定了一个未定义的内部操作 HostResolveImportedModule 的抽象加载机制， web 浏览器和 Node.js 可以自行决定用什么方式来实现 HostResolveImportedModule ， 以便更好地契合各自的环境







#### 在WEB浏览器中使用模块

在 script 标签中使用模块

``` html
// 加载一个module
<script type='module' src='./module.js'></script>
    
// 创建一个module
<script type='module'>
    import {sum} from './example.js'
    let result = sum(1,2)
</script>
```

此时 result 变量并未暴露在全局， 而只是在模块内部存在

> 当 type 属性无法辨认时， 浏览器会忽略  <script type='module'> 声明， 从而提供良好的向下兼容







#### web 浏览器中模块加载顺序

模块相对于脚本的独特之处在于， 他们能使用 import 来指定必须要加载的其让他文件， 以保证正确执行， 为了支持此功能， <script type='module'> 总是自动加上 defer 属性， defer 属性是加载脚本文件的可选项， 但在加载模块文件时总是自动应用的， 当 html 解析到有 src 属性的  <script type='module'> 标签是， 就会立即开始下载模块文件， 但并不会执行它，知道整个网页文档全部解析完位置， 模块也会按照他们在 html 文件中出现 的顺序依次执行， 这意味着 第一个  <script type='module'> 总是在第二个之前执行， 及时有些模块不用src 指定而是包含了内联脚本

``` html
//首先执行
<script type='module' src='./example.js' />

// 第二个执行
<script type='module'>
	let result = '123'
</script>

//最后执行
<script type='module' src='./example2.js' />
```

每个模块都可能用import 导入了一个或多个其他模块， 这就让事情变得复杂了， 这也就是为何模块会首先被解析， 因为这样才能识别出所有的import 语句， 每个import语句又会触发fetch ，并且在所有的import 导入的资源被加载与与执行完毕之前， 灭有任何模块会被执行

> 在前面的范例中， 完整的加载次序是 

1. 下载并解析 example.js
2. 递归下载并解析在 example.js 中使用 import 导入的资源
3. 解析内联模块
4. 递归下载并解析在内联模块中使用的import 导入的资源
5. 下载并解析 example2.js
6. 递归下载并解析在module2.js 中使用import 导入的资源

> 一旦加载完毕， 直到页面文档被完整解析之前， 都不会有任何代码被执行， 在文档解析完毕后 会发生下列行为

1. 递归执行module1 导入的资源
2. 执行 module1.js 
3. 递归执行内联模块导入的资源
4. 执行内联模块
5. 递归执行module2导入的资源
6. 执行module2.js

内联模块除了不必先现在代码之外， 与其他两个模块的行为一致， 加载import资源与执行模块的次序都是完全一样的

