### 冒泡

**将最小的数字一次冒出来**

网上的实现方式很多，我也写了一个，感觉更容易理解

```
function bubble(arr) {
    for (var i = 0; i < arr.length; i++) {
        var flag = true
        for (var j = arr.length - 1; j > i; j--) {
            if (arr[j] < arr[j - 1]) {
                var temp = arr[j-1]
                arr[j-1] = arr[j]
                arr[j] = temp
                flag = false
            }
        }
        if (flag) {
            break;
        }
    }
    return arr
}
console.log(bubble([100,3,44,1,5,2,50,9,6])) // [1, 2, 3, 5, 6, 9, 44, 50, 100]
```

整体思路如下所示：

![bubble](/imgs/bubble.gif)



[十大经典算法](https://mp.weixin.qq.com/s/vn3KiV-ez79FmbZ36SX9lg)，浅显易懂，拿走不谢