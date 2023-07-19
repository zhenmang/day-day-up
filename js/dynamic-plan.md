### 动态规划
通过研究「路径问题」可以学会「通用的解决 DP 的思路」。

![img](https://assets.leetcode.com/uploads/2018/10/22/robot_maze.png)

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

 

「示例 1」：

输入：m = 3, n = 7
输出：28


「示例 2」：

输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右
3. 向下 -> 向右 -> 向下


「示例 3」：

输入：m = 7, n = 3
输出：28


「示例 4」：

输入：m = 3, n = 3
输出：6



**js版本的实现**
```js
/**
 * @param {number} m
 * @param {number} n
 * @return {number}
 */
var uniquePaths = function(m, n) {
    const arr = Array(m).fill(Array(n).fill(1));
    arr[0][0] = 1;
    for (var i = 0; i < m; i++) {
        for (var j = 0; j < n; j++) {
            if (i===0 || j===0) {
                arr[i][j] = 1;
            } else {
                arr[i][j] = arr[i-1][j] + arr[i][j-1];
            }
        }
    }
    return arr[m-1][n-1];
};
```

解题总结：
1. 确定问题的状态定义
2. 确定状态转移方程
3. 前后状态相互依赖，知前可推后