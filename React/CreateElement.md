## CreateElement 解析

`React.createElement` 是 React 中用于创建元素的核心方法之一。它的主要作用是根据传入的参数生成一个 React 元素（即虚拟 DOM）



**函数的基本签名如下：**

`React.createElement(type, [props], [...children])`



#### 元素标签转义

当我们写React 标签时

``` jsx
<div id="foo">bar</div>

// Babel 会将其转义为

React.createElement('div', {id: 'foo'}, 'bar')
```





#### 组件转义

当我们写React 组件时

``` jsx
function Foo({id}) {
  return <div id={id}>foo</div>
}

<Foo id='foo'>
  <div id="bar">bar</div>
</Foo>

// Babel 会将其转义为
function Foo({id}) {
  return React.createElement('div', {id}, 'foo')
}

React.createElement(
	Foo,
  {id: 'foo'},
  React.createElement('div', {id: 'bar'}, 'bar')
)

```





#### 嵌套子元素转义

``` jsx
<div id='foo'>
	<div id='bar'>bar</div>
  <div id='baz'>baz</div>
</div>

// Babel 会将其转义为

React.createElement(
	'div',
  {id: 'foo'},
  React.createElement('div', {id: 'bar'}, 'bar'),
  React.createElement('div', {id: 'baz'}, 'baz')
)

// 其中 子元素是作为参数被追加到 createElement 中的
```





#### 源码精简版

``` jsx
function createElement(type, config, ...children) {
    let props = {};
    let key = null;
    let ref = null;

    // 处理 config 对象
    if (config != null) {
        // 提取 key 和 ref
        if (config.key !== undefined) {
            key = '' + config.key; // 将 key 转换为字符串
        }
        if (config.ref !== undefined) {
            ref = config.ref; // 获取 ref
        }

        // 将其他属性复制到 props 中
        for (let propName in config) {
            if (config.hasOwnProperty(propName) && !isReservedProp(propName)) {
                props[propName] = config[propName];
            }
        }
    }

    // 处理 children
    if (children.length === 1) {
        props.children = children[0]; // 如果只有一个子元素
    } else if (children.length > 1) {
        props.children = children; // 如果有多个子元素
    }

    // 返回一个 React 元素对象
    return {
        $$typeof: REACT_ELEMENT_TYPE, // 标识符，用于识别 React 元素
        type,
        key,
        ref,
        props,
    };
}
```



#### 总结

`React.createElement ` 创建的元素是一个虚拟 DOM 对象，它并不直接对应于浏览器中的 DOM 元素。React 使用这些虚拟 DOM 对象来高效地更新实际的 DOM。

虚拟 DOM 的使用使得 React 能够通过比较新旧虚拟 DOM 来最小化实际 DOM 的操作，从而提高性能。

创建的元素是不可变的，意味着一旦创建，就不能更改其 type 或 props。这有助于 React 在比较和更新时保持一致性。

不可变性使得 React 能够更高效地进行更新，因为它可以简单地比较引用而不是深度比较对象。

当你使用 JSX 语法时，实际上是通过 Babel 等工具将 JSX 转换为 `React.createElement` 调用。例如，`<div>Hello</div>` 会被转换为 `React.createElement('div', null, 'Hello')`。

这种转换使得开发者可以使用更直观的语法来创建元素，而不需要手动调用 `createElement`。

批量更新：React 在更新时会进行批量处理，减少不必要的渲染。`createElement` 的设计使得 React 能够在更新时高效地处理元素。

懒加载：在大型应用中，使用动态导入和懒加载可以减少初始加载时间，`createElement` 可以与这些技术结合使用，以优化性能。