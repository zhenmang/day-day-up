## 快速排序

1.在数组中选择一个元素作为基准点；

2.所有比基准值小的元素摆放在左边，而大于基准值的摆放在右边;

3.最后利用递归，将摆放在左边的数组和右边的数组重复进行上述的1和2操作。

![img](/imgs/)  



```
function quickSort(arr) {

  if (arr.length <= 1) {

    return arr;   

  }

  var pivotIndex = Math.floor(arr.length / 2);

  var pivot = arr.splice(pivotIndex, 1)[0];

  var left = [];

  var right = [];

  for (var i = 0; i < arr.length; i++) {

    if (arr[i] < pivot) {

      left.push(arr[i]);  

    } else {

      right.push(arr[i]);

    }

  }

  return quickSort(left).concat([pivot], quickSort(right))

}


quickSort([12,3,7,4,5,1,89])  // [1, 3, 4, 5, 7, 12, 89]
```

