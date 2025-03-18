## 从0搭建javascript库





#### 模块

传统模式中，没有模块的概念，需要凭借 `HTML` 文件中的 `script` 标签来引入 `JavaScript` 文件

随着项目规模变大，`script` 标签引入的其他库变多， 依赖关系变得复杂，被依赖的库必须先引入，变得难以维护



**AMD**

为浏览器设计的一种异步模块加载规范

>  todo ...



**CommonJS**

主要用于 `Node js` 环境中的同步模块加载规范

语法表达上，引入导出模块有差异

``` javascript
// 导出模块
function commonJsFunc() {}
moduel.exports = commonJsFunc

// 导入模块
const commonJsFunc = reuqire('....js')
```





**ES Module**

`ECMAScript 2015` 或 `ES6` ，目前一些浏览器已经支持 `ES Module` ，但是还需要构建工具来兼容更多的浏览器

语法表达上，引入导出模块上有差异

``` javascript
// 导出模块
export function esModuleFunc() {}

// 导入模块
import { esModuleFunc } from '.....js'
```





**UMD**

一种通用模块，是对其他模块规范的中和，可以在任何模块中使用（`AMD`， `Comonjs`, `ES Module`）





#### 适配不同模块规范







#### DEMO 目录结构

``` javascript
test-lib/
|
|- config/
|    |- rollup.config.aio.js
|    |- rollup.config.esm.js
|    |- rollup.config.js
|    |- rollup.js
|
|
|- dist/
|    |- index.aio.js 
|    |- index.esm.js
|    |- index.js
|  
|
|- src/  
|    |- index.js
|  
|
|- index.html  
|- index.js
|- package.json  
```







#### 打包步骤

以 `rollup` 为例

``` javascript
// 安装 rollup
$ npm i --save-dev rollup@0.57.1
```

创建配置文件，在根目录创建 `config` 文件夹，用于存放配置文件

* `config/rollup.config.js` 用于配置生成 `CommonJS ` 规范
* `config/rollup.config.esm.js` 用于配置生成 `ES Module` 规范
* `config/rollup.config.aio.js` 用于配置生成 `UMD ` 规范



``` javascript
// config/rollup.config.js 代码

module.exports = {
    input: 'src/index.js',
    output: {
        file: 'dist/index.js',
        format: 'cjs'
    }
}
```



``` javascript
// rollup.config.esm.js 代码

module.exports = {
    input: 'src/index.js',
    output: {
        file: 'dist/index.esm.js',
        format: 'es'
    }
}
```



执行打包命令

``` shell
$ npx rollup -c config/rollup.config.js
$ npx rollup -c config/rollup.config.esm.js
```

对于 `UMD ` 规范打包需要依赖 `rollup-plugin-node-resolve` 插件

``` shell
$ npm i --save-dev rollup-plugin-node-resolve@3.0.3
```

``` javascript
// rollup.config.aio.js 代码
var nodeResolve = require('rollup-plugin-node-resolve')

module.exports = {
    input: 'src/index.js',
    output: {
        file: 'dist/index.aio.js',
        format: 'umd',
        name: 'clone'
    },
    plugins: [
          nodeResolve({ nain: true, extensions: ['.js'] })
    ]
}
```

执行打包命令

``` javascript
$ npx rollup -c config/rollup.config.aio.js
```

此时 `dist` 目录下就生成了对应的代码

简化构建命令，配置在 `package.json` 文件中的 `scripts` 中

``` json
{
  "scripts": {
    "build:self": "rollup -c config/rollup.config.js",
    "build:esm": "rollup -c config/rollup.config.esm.js",
    "build:aio": "rollup -c config/rollup.config.aio.js",
    "build": "npm run build:self && npm run build:esm && npm run build:aio"
  }
}
```

目前为止，只需要 `npm run build` 即可构建全部

现在入口文件位于`dist` 目录下，需要修改 `package.json` ，指向 `dist`目录下的构建文件

``` json
{
    "main": "dist/index.js",
  	"jsnext:main": "dist/index.esm.js",
  	"module": "dist/index.esm.js"
}
```



编译后的代码如下

``` javascript
// dist/index.js

'use strict';

Object.defineProperty(exports, '__esModule', { value: true });

function clone(source) {
  var copyVal = JSON.parse(JSON.stringify(source));
  return copyVal;
}

exports.clone = clone;
```



``` javascript
// dist/index.esm.js

function clone(source) {
  var copyVal = JSON.parse(JSON.stringify(source));
  return copyVal;
}

export { clone };
```



``` javascript
// dist/index.aio.js

(function (global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
    typeof define === 'function' && define.amd ? define(['exports'], factory) :
    (factory((global.clone = {})));
}(this, (function (exports) { 'use strict';

    function clone(source) {
      var copyVal = JSON.parse(JSON.stringify(source));
      return copyVal;
    }

    exports.clone = clone;

    Object.defineProperty(exports, '__esModule', { value: true });

})));
```







#### 按需加载

> 参见 ：   https://cn.rollupjs.org/faqs/#what-is-tree-shaking





#### 兼容ES6

使用 `Babel` 将 `ES6` 代码转换为 `ES5` 代码, 针对 `rollup` 使用 `rollup-plugin-babel` 

``` shell
$ npm i --save-dev rollup-plugin-babel@4.0.3 @babel/core@7.1.2 @babel/preset-env@7.1.0
```

`Babel` 为每个 `ES6` 特性都提供了转换，这样开发者可以自寄决定要转换哪些特性，但是由开发者手动维护是困难的，所以推荐 `preset-env` , 会帮助开发者自动选择需要转换的特性，由于三个配置文件都需要配置 `preset-env` 所以把这个配置单独写在 `config/rollup.js` 中

``` javascript
// rollup.js
var babel = require('rollup-plugin-babel')

function getCompiler(opt) {
    return babel({
        babelrc: false,
        presets: [
            [
                '@babel/preset-env',
                {
                    targets: {
                        // browsers: 'last 2 versons, > 1%, ie >=8, chorme >= 45, safari >= 10',
                        ie: '8',
                        chorme: '45',
                        safari: '10',
                        node: '0.12'
                    },
                    modules: false,
                    loose: true
                }
            ]
        ],
        exclude: 'node_modules/**'
    })
}

exports.getCompiler = getCompiler
```

不单独使用 `Babel` 配置文件，所以 `babelrc` 和 `modules` 都设置为 `false`,  `loose` 代表松散模式 设置为 `true` 更好的兼容 `ie8`

然后  依次在三个配置文件中添加如下代码

``` javascript
var common = require('./rollup.js')

module.exports = {
    // ...
    plugins: [common.getCompiler()]
}
```

重新构建，`es6` 代码就被转换成了 `es5`





#### 单元测试

单元测试适合工具库的测试场景，提倡一边开发一边编写测试用例代码



**常见的测试框架**

* Mocha
* Jasmine
* Jest
* AVA



本次使用 `Mocha` 为例，需要安装 `Mocha` 和 `expect.js` 

``` shell
$ npm install --save-dev mocha@3.5.3
$ npm install --save-dev expect.js@0.3.1

$ mkdir test
$ touch test/test.js
```



`test.js` 中的代码如下：

``` javascript
// describe 来组织测试结构 可以嵌套
// describe 语义可以自定义
// it 代表一个测试用例
// 一个测试用例中可以有多个expect断言

var expect = require('expect.js')

describe(
	'单元测试',
  function() {
    describe(
    	'语义可嵌套',
      function() {
        
        it(
        	'hello',
          function() {
            expect(1).to.equal(1)
          }
        )
        
      }
    )
  }
)
```



执行测试，以下两种方式是等价的

``` shell
$ npx mocha
```

``` shell
$ ./node_modules/mocha/bin/mocha
```



将此命令添加到 `package.json` 文件的 `scripts` 中，然后通过 `npm test` 来执行测试

``` javascript
{
  "scripts": {
    "test": "mocha"
  }
}
```





#### 代码覆盖率

代码覆盖率可以反映单元测试的覆盖情况，衡量测试是否严谨的指标

`Istanbul` 是常用的检查覆盖率的库，提供的 `npm` 包叫做 `nyc`

``` shell
$ npm install --save-dev nyc@13.1.0
```

然后配置 `package.json` 中的 `scripts` 字段，然后 通过 `npm test` 来执行

``` javascript
{
  "scripts": {
    "test": "nyc mocha"
  }
}
```

执行 `npm test` 命令后，在命令行中会增加检查代码覆盖率的结果，其中：

* `Statement Coverage` - 语句覆盖率
* `Branch Coverage` - 分支覆盖率
* `Function Coverage` - 函数覆盖率
* `Line Coverage` - 行覆盖率

此外  `Istanbul` 提供输出多种格式的报告 其中可以使用留言器查看测试报告

``` shell
$ touch .nycrc
```

配置 `.nycrc` 文件

``` javascript
{
  "reporter": ["html", "text"]
}
```



修改 `src/index.js` 如下

``` javascript
export function clone(source) {
    const copyVal = JSON.parse(JSON.stringify(source))
    return copyVal
}

export function removeDuplication(a, b) {
    return [...new Set([...a, ...b])]
  }
```



重新执行命令 `npm test` 会在根目录生成 `coverage` 文件夹 打开其中的 `index.js.html`  就可以查案报告

![3336119db6515c90c26a406ba37b246e.png](https://i.miji.bid/2024/02/22/3336119db6515c90c26a406ba37b246e.png)

![43094196229585b3164aac34912394ca.png](https://i.miji.bid/2024/02/22/43094196229585b3164aac34912394ca.png)



#### 源代码覆盖率

上图展示的未被覆盖的代码是 `babel` 转换之后的代码，因为是测试 `dist` 目录下的代码 ， 源代码中有很多新特性的语法，低版本的 `Node js` 不支持

`Insanbul` 支持引入 `Babel` 进行构建 原理是先向源代码中插入测试代码覆盖率的代码，在进行 `Babel` 构建 将构建好的代码 交给 `mocha` 进行测试 这样就得到了源代码测试覆盖率

下面根据 `Istanbul` 官网提供的配置步骤修改测试流程

> 官方文档参见    https://github.com/istanbuljs/babel-plugin-istanbul

``` shell
$ npm i --save-dev @babel/register@7.0.0 babel-plugin-istanbul@5.1.0 cross-env@5.2.0
```

修改 `.nycrc` 文件 添加 `require` 配置 这样在 `test.js` 文件中通过 require引用的文件都会经过 `babel` 实时编译，使用 `babel`编译后就不需要  `nyc`  的 `sourceMap` 了, 对 源代码的覆盖率检测通过 `babel-plugin-istanbul` 实现 所以要将`instrument` 配置的值设置 `false`

``` javascript
{
  "require": ["@babel/register"],
  "reporter": ["html", "text"],
  "sourcemap": false,
  "instrument": false
}
```

上面配置 `rollup.js` 没有用独立的 `.babelrc` 配置文件, 所以要给`nyc` 单独添加一个`Babel` 配置文件 `.babelrc`  位于根目录

比之前增加了 `env.test.plugin.istanbul` 配置

使用 `plugin-istanbul `对源代码进行覆盖率测试

``` javascript
{
  "env": {
    "test": {
      "plugins": ["istanbul"]
    }
  }
}
```

上面配置的 `bebel-plugin-istanbul` 插件只有在环境变量中包含 `test` 时才会加载， 为了能跨平台使用， 可以通过 `croll-env` 来设置环境变量 , 修改 `package.json` 中的 `test` 字段

``` javascript
{
  "scripts": {
    "test": "cross-env NODE_ENV=test nyc mocha"
  }
}
```

最后 修改 `test/test.js` 中的代码， 引用 `src/index.js`

``` javascript
var clone = require('../dist/index.js').clone
// 改为
var clone = require('../src/index.js').clone
```

再次 执行 `npm test` 命令，查看代码覆盖率信息 发现已经成功执行了 源代码覆盖率

![截屏2024-02-22 下午2.17.50.png](https://img2.imgtp.com/2024/02/22/fkJY1jqk.png)



#### 校验覆盖率

> Todo...







#### 浏览器环境测试

目前单元测试代码只在 `Node js` 中运行 和浏览器环境不同 如果库中包含一些对浏览器操作 如 `DOM` 就会出现报错 导致无法运行





#### 模拟浏览器环境

在 `Node js` 中模拟 `DOM` 和 `BOM` 进行一些简单的测试  `jsdom` 是一种适合的方案

假设 有一个 获取 `url` 参数的函数 在 `src/index.js` 中 

``` javascript
// src/index.js

export function getUrlParam() {
  	const url = window.location.href
    return url
}
```

次方法依赖 `location` 但是 `node js` 中并没有此全局属性 因此可以使用 `jsdom` 模拟 需要安装 `mocha-jsdom`

``` shell
$ npm i --save-dev mocha-jsdom
```

然后修改单元测试代码

``` javascript
// test/test.js
var expect = require('expect.js')
var JSDOm = require('mocha-jsdom')
var clone = require('../src/index.js').clone
var getUrlParam = require('../src/index.js').getUrlParam

describe(
    '单元测试 clone',
    function () {
        var arr = [1, 2]
        var cloneArr = clone(arr)

        it('clone->', function () {
            expect(cloneArr).to.eql(arr)
        })
    }
)

describe(
    '单元测试 获取url',
    function () {
        JSDOM({ url: 'https://www.baidu.com/' })

        it('获取 url->', function () {
            expect(getUrlParam()).to.eql('https://www.baidu.com/')
        })
    }
)
```

执行 `npm run test`

![截屏2024-02-22 下午3.15.00.png](https://img2.imgtp.com/2024/02/22/Z6YyAZ9p.png)





#### 真实浏览器环境

通过`jsdom`只能模拟浏览器环境，单并不是真正的浏览器环境，`mocha`是支持浏览器环境中运行的

在 `test` 目录下添加 `browser/index.html` 文件

``` html
<!-- test/browser/index.html -->
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>mocha 浏览器环境</title>
    <link rel="stylesheet" href="../../node_modules/mocha/mocha.css" />
</head>

<body>
    <div id="mocha"></div>

    <script src="../../node_modules//mocha/mocha.js"></script>
    <script src="../../node_modules/expect.js/index.js"></script>
    <script src="../../dist/index.aio.js"></script>
  
		<!-- require 兼容代码 -->
    <script>
        var libs = {
            'expect.js': window['expect'],
            '../src/index.js': window['clone']
        }

        var require = function (path) {
            return libs[path]
        }
    </script>

    <script>
        mocha.setup('bdd')
    </script>

    <script src="../test.js"></script>

    <script>
        mocha.run()
    </script>
</body>

</html>
```

修改 `test/test.js `  注释掉 `JSDOM` 代码

``` javascript
var expect = require('expect.js')
// var JSDOM = require('mocha-jsdom')
var clone = require('../src/index.js').clone
var getUrlParam = require('../src/index.js').getUrlParam

describe(
    '单元测试 clone',
    function () {
        var arr = [1, 2]
        var cloneArr = clone(arr)

        it('clone->', function () {
            expect(cloneArr).to.eql(arr)
        })
    }
)

describe(
    '单元测试 获取url',
    function () {
        // JSDOM({ url: 'https://www.baidu.com/' })

        it('获取 url->', function () {
            // expect(getUrlParam()).to.eql('https://www.baidu.com/')
            expect(getUrlParam()).to.eql('file:///Users/01410136/Documents/test-project/my-lib/test/browser/index.html')
        })
    }
)
```

然后 浏览器打开 `test/browser/index.html`

![截屏2024-02-22 下午3.49.59.png](https://img2.imgtp.com/2024/02/22/CLtb1d51.png)





#### 自动化测试

> todo ...









#### 发布到 `github` 上

> todo ...









#### 发布到 npm上

先注册 npm账号 然后 

``` shell
$ npm login

$npm whoami # 查看是否登录成功（展示用户名）
```



配置黑名单 `.npmignore` 格式同 `.gitignore`

配置白名单 `package.json`

``` json
{
  "files": ["/dist", "LICENSE"]
}
```

如果黑名单和白名单同时存在 则会忽略黑名单

``` shell
$ npm pack --dry-run
```

此命令可以查看哪些文件会被发布, 通常来说 只需要发布 `dist` `LICENSE` 就可以

`README.md` `CHANGLOG.md` `package.json` 都是默认发布的



发布正式包和测试包

``` shell
# 测试包
$ npm public --tag=beta --access public

#正式包
$ npm public --access public
```



或者配置 `package.json` 之后 通过 `npm public` 命令发布

``` javascript
{
  "publicConfig": {
    "registry": "https:// registry.npmjs.org",
    "access": "public"
  }
}
```

