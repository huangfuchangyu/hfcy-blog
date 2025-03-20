#### Map

Map  方法创建一个新数组，其结果是该数组中的每个元素是调用一次提供的函数后的返回值。

``` javascript
const array1 = [1, 4, 9, 16];

// pass a function to map
const map1 = array1.map(x => x * 2);

console.log(map1);
// expected output: Array [2, 8, 18, 32]
```

### 

**参数**

- `callback`

  生成新数组元素的函数，使用三个参数：`currentValue``callback` 数组中正在处理的当前元素。`index`可选`callback` 数组中正在处理的当前元素的索引。`array`可选`map` 方法调用的数组。

- `thisArg`可选

  执行 `callback` 函数时值被用作`this`。



**返回值**

一个由原数组每个元素执行回调函数的结果组成的新数组。



**适用场景**

因为`map`生成一个新数组，当你不打算使用返回的新数组却使用`map`是违背设计初衷的，请用`forEach`或者`for-of`替代。你不该使用`map`的场景: 

A)你不打算使用返回的新数组，

B) 你没有从回调函数中返回值。









#### forEach()



forEach() 方法对数组的每个元素执行一次给定的函数。

``` javascript
const array1 = ['a', 'b', 'c'];

array1.forEach(element => console.log(element));

// expected output: "a"
// expected output: "b"
// expected output: "c"
```

### 

**参数**



- `callback`

  为数组中每个元素执行的函数，该函数接收一至三个参数：

  `currentValue`数组中正在处理的当前元素。`index` 可选数组中正在处理的当前元素的索引。`array` 可选`forEach()` 方法正在操作的数组。

- `thisArg` 可选

  可选参数。当执行回调函数 `callback` 时，用作 `this` 的值。

### 

**返回值**

[`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)。



**注意**

除了抛出异常以外，没有办法中止或跳出 `forEach()` 循环。如果你需要中止或跳出循环，`forEach()` 方法不是应当使用的工具。

若你需要提前终止循环，你可以使用：

- 一个简单的 [for](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for) 循环
- [for...of](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of) / [for...in](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in) 循环
- [`Array.prototype.every()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/every)
- [`Array.prototype.some()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/some)
- [`Array.prototype.find()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
- [`Array.prototype.findIndex()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)



**即 `forEach` 不会直接改变调用它的对象，但是那个对象可能会被 `callback` 函数改变**









#### for of