插件基本结构

``` javascript
class MyPlugin {
    constructor() { }

    apply(compiler) {

        compiler.hooks.run.tap("MyPlugin", () => console.log('开始编译...'))

        compiler.plugin('emit', function (compilation, callback) {
            
            callback()
        })
    }
}

module.exports = MyPlugin
```



里面包含 `apply` 原型方法

在使用插件 时 配置如下： 

``` javascript
plugins: [
    new MyPlugin({ options: true })
]
```

安装插件时， 在 `webpack config` 里 配置 `plugin`

1. `webpack` 启动 读取配置文件， 在 `plugin` 数组内 实例化 插件 
2. 通过 插件 实例， 调用 `apply` 方法， 传入 `compiler` 对象
3.  在 `apply`方法内  通过  传入的 `compiler` 对象 注册 `tapable` 的 `hook` 回调
4. 在 `tapable` 的 `hook` 回调中 操作 模块



其中 

`compiler` 表示 整个 `webpack` 从启动 到 结束的生命周期

`compilation` 表示一次 新的编译， 比如 文件跟新 `compilation` 重新创建