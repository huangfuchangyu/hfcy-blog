## tree-shaking

用途是 消除工程中的 DCE（Dead Code Elimination）

* 代码不会被执行， 不可到达
* 代码执行的结果不会被用到
* 代码只会影响到死变量（只写不读）

webpack 识别标记 DCE ，  压缩插件 把 DCE 打掉



`tree-shaking` 利用 ES 模块的 静态 import 特性， 和 运行无关， 进行可靠的静态分析， 这是 `tree-shaking`识别 ECE 的基础



副作用是 对于  全局样式  或者 配置文件的处理 

``` javascript
import 'xxxx.css'
import 'xxxx.js'  // 自执行
```

对于 这种 文件 ， 要配置 `sideEffect`， 不进行 `tree-shaking`

webapck 要把 `bable`  `module=false`

用`ESM` 这是 `tree-shaking` 的前提， 否则 会被编成 `commonjs`来执行

