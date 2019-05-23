## react 生命周期

## react key 作用

    key属性的使用，则涉及到diff算法中同级节点的对比策略，当我们指定key值时，
    key值会作为当前组件的id，diff算法会根据这个id来进行匹配
    

## react diff 策略

    Web UI中DOM节点跨层级的移动操作特别少，可以忽略不计。
    拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构
    对于同一层级的一组子节点，它们可以通过唯一 id 进行区分

    在上面三个策略的基础上，React 分别将对应的tree diff、component diff 以及 element diff 进行算法优化，极大地提升了diff效率

    1. tree diff
        既然 DOM 节点跨层级的移动操作少到可以忽略不计，针对这一现象，React只会对相同层级的 DOM 节点进行比较，
        即同一个父节点下的所有子节点。当发现节点已经不存在时，则该节点及其子节点会被完全删除掉，不会用于进一步的比较。
        这样只需要对树进行一次遍历，便能完成整个 DOM 树的比较
        (在开发组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，可以通过 CSS 隐藏或显示节点，而不是真正地移 除或添加 DOM 节点。)

    2. component diff

        如果是同一类型的组件，按照原策略继续比较 Virtual DOM 树即可

        如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点

        对于同一类型的组件，有可能其 Virtual DOM 没有任何变化，如果能够确切知道这点，
        那么就可以节省大量的 diff 运算时间。因此，React允许用户通过shouldComponentUpdate()
        来判断该组件是否需要进行diff算法分析，但是如果调用了forceUpdate方法，shouldComponentUpdate则失效

    3. element diff
        对同一层级的同组子节点，添加唯一 key 进行区分

        新旧集合中的节点都是相同的节点时

        如果新集合中有新加入的节点且旧集合存在时

        对于简单列表页渲染来说，不加key要比加了key的性能更好
        key的作用： 准确判断出当前节点是否在旧集合中   极大地减少遍历次数


## mobx-react 如何驱动 react 组件重渲染

## react 组件间通信

    父组件向子组件通信：使用 props
    子组件向父组件通信：使用 props 回调
    跨级组件间通信：使用 context 对象
    非嵌套组件间通信：使用事件订阅


## react mixin 和 HOC 区别


## react pureComponent 适用场景

    PureComponent的原理是继承了Component类，自动加载shouldComponentUpdate函数，当组件更新时，shouldComponentUpdate对props和state进行了一层浅比较，如果组件的props和state都没有发生改变，render方法就不会触发，省去Virtual DOM的生成和对比过程，达到提升性能的目的

    所以想要用PureComponent的特性，应该遵守原则：

    确保数据类型是值类型
    如果是引用类型，不应当有深层次的数据变化(解构)

## vuex, mobx, redux 各自的特点和区别