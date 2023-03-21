### 二分查找

**在一个有序数组arr中，找出目标值dest所在的位置**

1.取两个位置，最左边和最右边，分别记为start、end；

2.然后取两个位置的中间位置middle = Math.floor(（start + end）/ 2 )；

3.比较中间位置的值与目标数据的大小，若 目标数据 > 中间位置的值，则将左位置start向右移动为middle+ 1；若目标数据 < 中间位置的值，则将右位置end向左移动为middle - 1，若相等，则为目标值

4.如此循环，直至找到目标值。



```
/**
* @param {Array} arr
* @param {*} dest
*/
var binarySearch = function (arr, dest) {
  var start = 0
  var end = arr.length - 1
  while (start <= end) {
    var middle = Math.floor((start + end) / 2)
    if (dest > arr[middle]) {
      start = middle + 1
    } else if (dest < arr[middle]) {
      end = middle - 1
    } else {
      return middle
    }
  }
}

var arr = [-34, 1, 3, 4, 5, 8, 34, 45, 65, 87]
console.log(binarySearch(arr, 65)) // 8
console.log(arr[8]===65) // true
```

![half-lookup](/imgs/half-lookup.jpg)