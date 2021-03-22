`webapck` 只能 编译 `javascript` 和 `json`  ， `loader` 能让 `webpack` 去 编译  识别不了的文件类型，进行 文件转换 ， 添加到 依赖中

本质上说 `loader` 是一个 `node` 模块 ， 是一个函数， `loader`  会在 转换 源模块 的时候 调用 该函数

在这个 函数内部， 可以通过 `thsi`上下文 ， 操作  `loader api` 最终 转换成 可用 的模块

```javascript
module.exports = function (source, sourceMap) {
    return source.replace(/var/g, 'const')
}
```

配置方法

``` javascript
module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: path.resolve('./Myloader.js'),
          },
        ]
      }
    ]
  }
```

`loader` 支持 链式调用， 如果 把`loader` 的结果 传给 下一个 `loader`  则 需要 `this.callback(null,param)`

其中 `callback` 函数签名为

```javascript
this.callback(err, content, sourceMap)
```

如果 loader 内  是 同步 操作  ， 直接   `return ` 处理后的结果就可以

如果是异步操作 ， 则 需要调用 `this.callback()` 



`loader` 开发准则

* 单一职责
* 模块化
* 无状态





