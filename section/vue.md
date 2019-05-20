## vue 生命周期



## vue 组件间通信

    父组件向子组件通信
        1. props
        2. $children

    子组件向父组件通信
        1. 使用vue事件
        2. 过修改父组件传递的props来修改父组件数据(父组件传递一个引用变量时可以使用)
        3. $parent
    
    非父子组件、兄弟组件之间的数据传递
        1. Vue官方推荐使用一个Vue实例作为中央事件总线
        eg: 
            $on方法用来监听一个事件。
            $emit用来触发一个事件。
            /*新建一个Vue实例作为中央事件总嫌*/
            let event = new Vue();

            /*监听事件*/
            event.$on('eventName', (val) => {
                //......do something
            });

            /*触发事件*/
            event.$emit('eventName', 'this is a message.');

    复杂的单页应用数据管理
        vuex


## Vue 响应式原理

    把一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，
    Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 
    把这些属性全部转为 getter/setter
    每个组件实例都对应一个 watcher 实例，
    它会在组件渲染的过程中把“接触”过的数据属性记录为依赖。
    之后当依赖项的 setter 触发时，会通知 watcher，从而使它关联的组件重新渲染
    Vue 无法检测到对象属性的添加或删除。
    由于 Vue 会在初始化实例时对属性执行 getter/setter 转化，
    所以属性必须在 data 对象上存在才能让 Vue 将它转换为响应式的
    对于已经创建的实例，Vue 不允许动态添加根级别的响应式属性。
    但是，可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式属性
    Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，
    并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次
    这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。
    然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作
    当你设置 vm.someData = 'new value'，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新
    为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 Vue.nextTick(callback)。这样回调函数将在 DOM 更新完成后被调用


## vue 的 nextTick 实现原理以及应用场景

    Vue 是异步执行 DOM 更新的
    所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）
    主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件
    一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行
    主线程不断重复上步
    Vue 在修改数据后，视图不会立刻更新，而是等同一事件循环中的所有数据变化完成之后，再统一进行视图更新

    当数据改变时
    第一个 tick 
        1. 首先修改数据，这是同步任务。同一事件循环的所有的同步任务都在主线程上执行，形成一个执行栈，此时还未涉及 DOM
        2. Vue 开启一个异步队列，并缓冲在此事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次
    第二个 tick
        同步任务执行完毕，开始执行异步 watcher 队列的任务，更新 DOM 。Vue 在内部尝试对异步队列使用原生的 Promise.then 和 MessageChannel 方法，
        如果执行环境不支持，会采用 setTimeout(fn, 0) 代替
    第三个 tick
        下次 DOM 更新循环结束之后
        此时通过 Vue.nextTick 获取到改变后的 DOM 。通过 setTimeout(fn, 0) 也可以同样获取到。

    需要注意的是，在 created 和 mounted 阶段，如果需要操作渲染后的试图，也要使用 nextTick 方法
    官方文档说明: 
        注意 mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.$nextTick 替换掉 mounted
        mounted: function () {
            this.$nextTick(function () {
                //  ... do something
            })
        }


## vue comput watch 区别


##  为什么 Vue 无法探测根数据节点普通的新增属性 

    Object.definProperty()

## vue 如何 探测根数据节点普通的新增属性 

    vm.$set()