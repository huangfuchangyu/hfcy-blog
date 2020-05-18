## SetState()



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

