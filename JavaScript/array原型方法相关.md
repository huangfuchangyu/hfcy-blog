#### Array 原型方法相关



**其中 不会改变 原数组的方法有 **

* Array.from()
* Array.concat()
* Array.slice()
* Array.toString()



###### Array.from()   

>  --  返回新数组， 数组项为 浅拷贝

从 一个 类数组或 可迭代对象创建一个新的, 浅拷贝的数组

``` javascript
let arr = [1,2,3]
let arr2 = Array.from(arr)

console.log(arr2)
VM449:3 (3) [1, 2, 3]

arr.push(4)

console.log(arr)
VM526:1 (4) [1, 2, 3, 4]

console.log(arr2)
VM574:1 (3) [1, 2, 3]
```



###### Array.concat()  

>  -- 返回新数组

合并两个或多个数组， 不会改变现有数组， 而是返回新数组

``` javascript
let arr1 = [1,2,3]
let arr2 = [4,5,6]
let arr3 = arr1.concat(arr2)

console.log(arr3)
VM1063:4 (6) [1, 2, 3, 4, 5, 6]

arr1.push(0)
arr2.push(9)

console.log(arr3)
VM1181:3 (6) [1, 2, 3, 4, 5, 6]
```



###### Array.pop()

> 此方法从数组中删除最后一个元素， 并返回该元素值， 此方法改变数组长度

``` javascript
let arr1 = [1,2,3]
let arr2 = arr1.pop()

console.log(arr1)
VM1729:3 (2) [1, 2]
```





###### Array.push()

> 此方法将一个或多个元素添加到数组末尾， 并返回该数组的 新长度 

``` javascript

```





###### Array.reverse()

> 将数组中的元素的位置颠倒，并返回该数组， 该方法会改变原数组

``` javascript
let arr1 = [1,2,3,4]
let arr2 = arr1.reverse()

console.log(arr2)
VM1961:4 (4) [4, 3, 2, 1]

arr1.push(5)

console.log(arr2)
VM2056:2 (5) [4, 3, 2, 1, 5]
```





###### Array.shift()

> 删除数组中第一个元素， 并返回该元素的值， 此方法更改数组长度







###### Array.slice()

> 方法返回一个新的数组对象， 这个对象是由begin 和 end 决定的 原数组的浅拷贝（包括 begin 不包括 end ） 原始数组不会改变

``` javascript
let arr = [1,2,3,4,5]
let arr2 = arr.slice(0,2)

console.log(arr)
console.log(arr2)

VM2541:4 (5) [1, 2, 3, 4, 5]
VM2541:5 (2) [1, 2]
```





###### Array.sort()

> 采用 原地算法 对数组进行排序， 并返回数组， 默认排序顺序是在将元素转换为字符串 ， 然后比较他们的UTF-16代码单元值序列时构建的， 会改变原数组

``` javascript
let nums = [3,2,4,5,1]
nums.sort((a,b) => a-b )

console.log(nums)
VM3111:4 (5) [1, 2, 3, 4, 5]
```





###### Array.splice()

> 此方法通过 删除或替换现有元素 或者 原地添加新元素来修改数组， 并以数组形式返回被修改内容， 此方法会改变原数组

``` javascript
let arr = [1,2,3,4,5]
arr.splice(2, 0, 'aa')

console.log(arr)
VM3283:4 (6) [1, 2, "aa", 3, 4, 5]
```





###### Array.toString()

> toString() 返回一个字符串， 表示指定的数组及元素， 不会改变原数组

``` javascript
let arr = [1,2,3]
let arr2 = arr.toString()

console.log(arr)
console.log(arr2)

VM230:4 (3) [1, 2, 3]
VM230:5 1,2,3
```





###### Array.unshift()

> 将 一个或多个元素添加到数组的开头， 并返回该数组的新长度， 该方法会改变原有数组

``` javascript
let arr = [1,2,3]
arr.unshift(0)

console.log(arr)
VM385:4 (4) [0, 1, 2, 3]
```

