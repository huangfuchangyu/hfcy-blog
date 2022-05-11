## 如何编写eslint 插件



#### eslint 安装

1. Vscode 安装 eslint 插件
2. Npm 安装 eslint  `npm install eslint --save-dev`
3. Eslint 初始化 `eslint --init` 或者 `./node_modules/.bin/eslint --init` 

  初始化完 vscode 右下角 会展示  ESLINT 按钮

  可以先配置一条规则 判断一下是否生效（需要重启 vscode）



#### 创建 插件

使用 [Yeoman generator](https://www.npmjs.com/package/generator-eslint) 创建，它将引导你完成插件框架的设置

`npm i generator-eslint`

目录结构：

```
|-- lib
	 |-- rules
|-- index.js
|-- node_modules
|-- .eslint.js
|-- package.json
|-- README.md
```

导出配置

``` jsx
// index.js
const output = {
  configs: {
    // 导出自定义规则 在项目中直接引用
    myConfig: require('./rules/simple.js')
  }
}
module.exports = output;
```

```jsx
// simple.js
module.exports = {
    plugins: ["myplugin"],
    rules: {
        "no-console": 2,
        'no-var': 'error'
    }
}
```



然后 配置 package.json 版本

`npm pack` 打包

在 使用的工程中 `.eslintrc.js` 

``` javascript
module.exports = {
    "env": {
        "browser": true,
        "es2021": true
    },
    "extends": [
        "plugin:myplugin/myConfig"
    ],
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "ecmaVersion": "latest",
        "sourceType": "module"
    },
    "plugins": [
        "myplugin"
    ],
    "rules": {
    }
}

```

其中  要配置  `plugins` 和 `extends` 



#### 自定义lint 规则

