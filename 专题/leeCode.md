#### 两数之和

>给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。
>
>你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍
>

``` javascript
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1]
```

解法1：

``` javascript
// 双重循环 
略
```

解法2：

``` javascript
// 循环一次  用空间 换时间
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {

    let hasLoopMap = new Map()

    hasLoopMap.set(nums[0], 0)

    for (let i = 1; i < nums.length; i++) {

        const diff = target - nums[i]

        if (hasLoopMap.has(diff)) {
            return [hasLoopMap.get(diff), i]
        }
        else {
            hasLoopMap.set(nums[i], i)
        }
    }
};
```





#### 找出数组中重复的数字

>在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字

``` javascript
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

解法： 

``` javascript
var findRepeatNumber = function (nums) {

    let map = new Map()

    for (const item of nums) {
        if(map.get(item)) {
            return item
        }
        map.set(item, 1)
    }

}
```





#### 二维数组中查找

>在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

``` javascript
示例:

现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]

给定 target = 5，返回 true。

给定 target = 20，返回 false。
```

解法： 

``` javascript
// 螃蟹走法   以 左下角 或者 右上角 为 起点
// 与 当前 值 对比 ， 通过 大小 来 + _ 坐标位置 

/**
 * @param {number[][]} matrix
 * @param {number} target
 * @return {boolean}
 */
var findNumberIn2DArray = function (matrix, target) {

    const maxRow = matrix.length
    const maxColumn1 = matrix[0]

    let curRow = maxRow - 1
    let curColumn = 0

    while (curRow >= 0 && curColumn < maxColumn1.length) {

        if (matrix[curRow][curColumn] === target) {
            return true
        }
        else if (matrix[curRow][curColumn] > target) {
            curRow--
        }
        else {
            curColumn++
        }
    }

    return false
}
```

