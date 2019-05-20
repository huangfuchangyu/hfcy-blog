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

