## SetState()



调用 setState() 时  ， 实际上发生了 

1. 调用 enqueueSetState（） 方法  并且 传递了 实例 和状态
2. 将要更新的状态 放在  internalInstance._pendingStateQueue  数组里
3.  将 internalInstance._pendingStateQueue 数组   传给 enqueueUpdate（） 处理
4. 判断 BatchingUpdates 状态  如果 正在 BatchingUpdates  执行 5， 否则 执行 6
5. 将当前 组件 放在 dirtyComponent 中 
6.  将 BatchingUpdates  状态更新为 true， 循环 dirtyComponent  中的 updateComponent 来更新组件



**注： **

1. 由React控制的事件处理程序，以及生命周期函数调用setState不会同步更新state 。
2. React控制之外的事件中调用setState是同步更新的。比如原生js绑定的事件，setTimeout/setInterval等。



setState 源码

``` javascript
/**
 * @param {object|function} partialState Next partial state or function to
 *        produce next partial state to be merged with current state.
 * @param {?function} callback Called after state is updated.
 */
ReactComponent.prototype.setState = function(partialState, callback) {
    
  // ...
    
  this.updater.enqueueSetState(this, partialState);
    
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
    
}
```

重点  enqueueSetState



enqueueSetState源码

``` javascript
  enqueueSetState: function(publicInstance, partialState) {
    
    // ...  
      	
    // 获取 调用了 setSate 的 组件的 实例  
    var internalInstance = getInternalInstanceReadyForUpdate(
      publicInstance,
      'setState',
    );

    if (!internalInstance) {
      return;
    }

    var queue =
      internalInstance._pendingStateQueue ||
      (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

    enqueueUpdate(internalInstance);
  },
      
      

```

其中 getInternalInstanceReadyForUpdate 实现：

``` javascript
      
    function getInternalInstanceReadyForUpdate(publicInstance, callerName) {
  		var internalInstance = ReactInstanceMap.get(publicInstance);
  		// ...
 		return internalInstance;
	}
```

重点关注  enqueueUpdate  :  

``` javascript
function enqueueUpdate(component) {
  
  // ...
	
  // 此处 有个判断  是否在 BatchingUpdates 状态 
  // 如果 在 BatchingUpdates 状态 把当前 component 放到 dirtyComponents 队列中
  // 如果 不在 BatchingUpdates 状态  ， 执行 batchedUpdates（）
  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  dirtyComponents.push(component);
  if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
}
```

那么  什么时候 isBatchingUpdates 为  true 什么时候  为 false 呢 

其中 batchingStrategy 的定义 为 

``` javascript
var ReactDefaultBatchingStrategy = {
  isBatchingUpdates: false,

	
  batchedUpdates: function(callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;
	
    // 在这里  当 执行 batchedUpdates 时  isBatchingUpdates 为 true
    ReactDefaultBatchingStrategy.isBatchingUpdates = true;

    // The code is written this way to avoid extra allocations
    if (alreadyBatchingUpdates) {
      return callback(a, b, c, d, e);
    } else {
      return transaction.perform(callback, null, a, b, c, d, e);
    }
  },
};
```

这里 我们知道 当 执行 batchedUpdates 为  true 

什么时候为 false 呢  在这里 

``` javascript
var RESET_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: function() {
    ReactDefaultBatchingStrategy.isBatchingUpdates = false;
  },
};
```

问题来了  , 什么时候 会 执行 RESET_BATCHED_UPDATES.close(),   继续 找源码

``` javascript
// TODO 
```





继续看 ReactDefaultBatchingStrategy.batchedUpdates() 方法 ， 此方法执行了 transaction.perform（）

transaction 是 ReactDefaultBatchingStrategyTransaction 的一个实例

``` javascript
var transaction = new ReactDefaultBatchingStrategyTransaction()
```

