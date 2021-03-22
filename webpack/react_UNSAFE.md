react 新版本生命周期中， 对于 旧的 生命周期函数

* UNSAFE_componentWillMount()
* UNSAFE_componentWIllUpdate()
* UNSAFE_componentWillReceiveProps()

前面加了 UNSAFE 前缀



新增了

* getSnapshotBeforeUpdate()
* static getDesivedStateFromProps()





**UNSAFE_componentWillMount**

在 react 新版本中， 异步渲染机制可能 导致 willMount 被调用多次 ， 如果 在 willMount 中 做一些 副操作（订阅， 网络请求等）， 可能会导致 触发多次



**UNSAFE_componentWIllUpdate**

在 旧的版本中 ， willUpdate 一般配合 didUpdate 一起使用， 异步渲染 可能导致 两个回调 时间差距 过长， 如果 中间 进行一些 IO 操作等 改变了 状态 ， 会导致 问题 



**getSnapshotBeforeUpdate**

为了解决 willUpdate  和 didUpdate  间 状态 不统一的问题， getSnapshotBeforeUpdate 发生在 更新 dom 之前