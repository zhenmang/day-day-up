### 合并区间
以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间 。

「示例 1」：

输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].


「示例 2」：

输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。



JavaScript实现
```js
/**
 * @param {number[][]} intervals
 * @return {number[][]}
 */
var merge = function(intervals) {
    if (!intervals || !intervals.length || intervals.length === 1) {
        return intervals;
    }
    intervals.sort((a, b) => {
        return a[0] === b[0] ? (a[1] - b[1]) : (a[0] - b[0]);
    })
    var result = [];
    intervals.reduce((cur, next, index, arr) => {
        if (cur[1] >= next[1]) {
        } else if (cur[1] >= next[0]) {
            cur = [cur[0], next[1]];
        } else {
            result.push(cur);
            cur = next;
        }
        if (index === arr.length - 1) {
            result.push(cur);
        }
        return cur;
    });
    return result;
};
```