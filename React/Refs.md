## Refs 和 forwardRef



**汇总：**

在 React 16 版本（特别是 16.3 之前的版本）中，由于缺乏 `forwardRef` 原生支持和旧的 ref 处理机制，存在多个与 ref 传递相关的典型问题

* 高阶组件（HOC） 中的 Ref 丢失问题

``` jsx
// HOC 实现
function withLogging(WrappedComponent) {
  return class extends React.Component {
    render() {
      return <WrappedComponent {...this.props} />;
    }
  };
}

// 使用 HOC 的组件
class MyInput extends React.Component {
  render() {
    return <input ref={this.props.inputRef} />;
  }
}

const EnhancedInput = withLogging(MyInput);

// 父组件调用
class Parent extends React.Component {
  inputRef = React.createRef();

  render() {
    // ref 实际指向的是 HOC 实例而非 MyInput
    return <EnhancedInput inputRef={this.inputRef} />;
  }
}
```



* 函数组件的Ref 黑洞问题

函数组件无法直接绑定 ref，需要配合 `React.forwardRef`（16.3+）或使用类组件

``` jsx
// 函数组件无法接收 ref
function FunctionalComponent() {
  return <div />;
}

class Parent extends React.Component {
  ref = React.createRef();

  componentDidMount() {
    console.log(this.ref.current); // null（React 16.3 之前）
  }

  render() {
    return <FunctionalComponent ref={this.ref} />;
  }
}
```



* 条件渲染导致Ref 错乱

**Bug 表现**：在 React 16.4 之前，切换条件分支时可能发生旧 ref 未及时清除的问题

``` jsx
class DynamicComponent extends React.Component {
  state = { show: true };

  refA = React.createRef();
  refB = React.createRef();

  toggle = () => {
    this.setState({ show: !this.state.show });
  };

  render() {
    return (
      <>
        <button onClick={this.toggle}>Toggle</button>
        {this.state.show ? (
          <Child key="A" ref={this.refA} />
        ) : (
          <Child key="B" ref={this.refB} />
        )}
      </>
    );
  }
}
```



* 列表渲染中的 Ref 索引错位

**风险**：当列表项顺序变化时（如排序、过滤），`refsArray` 的索引与 DOM 节点无法保持同步

``` jsx
class List extends React.Component {
  items = [{ id: 1 }, { id: 2 }];
  refsArray = this.items.map(() => React.createRef());

  render() {
    return (
      <ul>
        {this.items.map((item, index) => (
          <li key={index} ref={this.refsArray[index]}>
            {item.id}
          </li>
        ))}
      </ul>
    );
  }
}
```



* 字符串 Ref 的内存泄露风险

``` jsx
class LegacyComponent extends React.Component {
  componentDidMount() {
    // 通过字符串 ref 访问
    console.log(this.refs.myRef);
  }

  render() {
    return <div ref="myRef" />;
  }
}
```





#### 三种使用  Ref 的方式

下面三种方式只能适用于 class 组件， 不能用于 函数组件， 因为函数组件没有实例

> Refs 除了用于获取具体的 DOM 节点外，也可以获取 Class 组件的实例，当获取到实例后，可以调用其中的方法，从而强制执行，比如动画之类的效果

**String Refs**

``` jsx
class App extends React.Component {
    constructor(props) {
        super(props)
    }
    componentDidMount() {
        setTimeout(() => {
             // 2. 通过 this.refs.xxx 获取 DOM 节点
             this.refs.textInput.value = 'new value'
        }, 2000)
    }
    render() {
        // 1. ref 直接传入一个字符串
        return (
            <div>
              <input ref="textInput" value='value' />
            </div>
        )
    }
}

root.render(<App />);
```



**回调Refs**

``` jsx
class App extends React.Component {
    constructor(props) {
        super(props)
    }
    componentDidMount() {
        setTimeout(() => {
              // 2. 通过实例属性获取 DOM 节点
              this.textInput.value = 'new value'
        }, 2000)
    }
    render() {
        // 1. ref 传入一个回调函数
        // 该函数中接受 React 组件实例或 DOM 元素作为参数
        // 我们通常会将其存储到具体的实例属性（this.textInput）
        return (
            <div>
              <input ref={(element) => {
                this.textInput = element;
              }} value='value' />
            </div>
        )
    }
}

root.render(<App />);
```



**`createRef`**

``` jsx
class App extends React.Component {
    constructor(props) {
        super(props)
        // 1. 使用 createRef 创建 Refs
        // 并将 Refs 分配给实例属性 textInputRef，以便在整个组件中引用
        this.textInputRef = React.createRef();
    }
    componentDidMount() {
        setTimeout(() => {
            // 3. 通过 Refs 的 current 属性进行引用
            this.textInputRef.current.value = 'new value'
        }, 2000)
    }
    render() {
        // 2. 通过 ref 属性附加到 React 元素
        return (
            <div>
              <input ref={this.textInputRef} value='value' />
            </div>
        )
    }
}
```

`cerateRef` 简化后源码如下

``` javascript
export function createRef() {
  const refObject = {
    current: null,
  };
  return refObject;
}
```





#### Refs 转发

对于“获取组件实例，调用实例方法”这个需求，即使使用 forwardRef 也做不到，借助 forwardRef 后，我们也就是跟类组件一样，可以在组件上使用 ref 属性，然后将 ref 绑定到具体的 DOM 元素或者 class 组件上，也就是我们常说的 Refs 转发。





#### `forwardRef`



forwardRef的主要作用是允许父组件访问子组件的DOM节点或组件实例。在函数组件中，默认不能直接使用ref，因为函数组件没有实例。forwardRef通过将ref传递到子组件内部来解决这个问题。



`forwardRef` 源码如下： 

``` typescript
export function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
  // 开发环境校验
  if (__DEV__) {
    if (render != null && render.$$typeof === REACT_MEMO_TYPE) {
      console.error('forwardRef requires a render function but received a `memo` ' + 'component...');
    }
  }

  const elementType = {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render, // 保存传入的 render 函数
  };
  
  // 开发环境下添加 displayName
  if (__DEV__) {
    Object.defineProperty(elementType, 'displayName', {
      enumerable: false,
      configurable: true,
      get: function() {
        return render.displayName || render.name || '';
      },
      set: function(name) {},
    });
  }

  return elementType;
}
```



使用示例

``` jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="fancy-button">
    {props.children}
  </button>
));

const App = () => {
  const buttonRef = React.useRef(null);

  return <FancyButton ref={buttonRef}>Click Me</FancyButton>;
};
```

