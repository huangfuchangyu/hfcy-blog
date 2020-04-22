## React 首屏加载优化



* webpack-bundle-analyzer

  使用此插件  查看 代码中 各个 依赖的大小 以便于 找出 需要优化的依赖项

* 一些大的依赖采用按需引入  如 （antd  echarts）

* 采用代码分割 

  ``` javascript
  //  1.  采用动态 import 来拆包
  import("./math").then(math => {
    console.log(math.add(16, 26));
  })
  
  // 当 Webpack 解析到该语法时，会自动进行代码分割。如果你使用 Create React App，该功能已开箱即用
  // 如果你自己配置 Webpack，你可能要阅读下 Webpack 关于代码分割的指南
  // https://webpack.docschina.org/guides/code-splitting/
  
  
  // 2. React.lazy  组件懒加载
  
  const OtherComponent = React.lazy(() => import('./OtherComponent'))
  
  // 此代码将会在组件首次渲染时，自动导入包含 OtherComponent 组件的包
  // React.lazy 接受一个函数，这个函数需要动态调用 import()。它必须返回一个 Promise，该 Promise 需要 resolve 一个 defalut export 的 React 组件
  // 然后应在 Suspense 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）
  
  //<React.Suspense fallback={<div>Loading...</div>}>
  //    <OtherComponent />
  // </React.Suspense>
  
  // fallback 属性接受任何在组件加载过程中你想展示的 React 元素。你可以将 Suspense 组件置于懒加载组件之上的任何位置。你甚至可以用一个 Suspense 组件包裹多个懒加载组件
  
  ```

  

* 采用 骨架图 来一定程度的避免白屏时间
* 布置生产版本