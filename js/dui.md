### **概念**

堆的底层实际上是一棵完全二叉树，可以用数组实现。

二叉树的一种，满足以下条件：

1. 任意节点大于或小于它的所有子节点（大根堆、小根堆）
2. 总是一完全树，即除了最底层，其它层的节点都被元素填满

将根节点最大的堆叫做`最大堆`或`大根堆`，根节点最小的堆叫做`最小堆`或`小根堆`。

![img](/imgs/paixuweizhi.png)

将数组`第一个元素置空`，为了方便计算。这样我们就可以从下标`1`开始，下标变量为`i`,那么：

- 左子节点位置是 `2*i`
- 右子节点位置是 `2*i+1`
- 父节点位置是 `Math.floor(i/2)`

堆（以大顶堆为例）相关的操作主要有：

1. **大顶堆调整**（Max-Heapify），将堆的末端子节点做调整，使得子节点永远小于父节点；
2. **创建大顶堆**（Build-Max-Heap），将堆中所有数据调整位置，使其成为大顶堆；
3. **堆排序**（Heap-Sort），移除在堆顶的根节点，并做大顶堆调整的迭代运算。

### **大顶堆调整**

```text
/**
 * 从index开始检查并保持大顶堆
 * @param arr  待检查数组
 * @param i    检查的起始下标
 * @param size 堆大小
 */
function maxHeapify(arr, i, size) {
  let left = 2 * i
  let right = left + 1

  //左右孩子中较大的一个
  let maxlr = -1

  //无左右孩子节点
  if (left > size && right > size) {
    return
  }
  //只有左孩子节点
  if (left <= size && right > size) {
    maxlr = left
  }
  //只有右孩子节点
  if (right <= size && left > size) {
    maxlr = right
  }
  //同时有左右孩子节点
  if (left <= size && right <= size) {
    maxlr = arr[left] < arr[right] ? right : left
  }

  if (arr[i] < arr[maxlr]) {
    swap(arr, i, maxlr)
    maxHeapify(arr, maxlr, size)
  }
}

function swap(arr, i, j) {
  let temp = arr[i]
  arr[i] = arr[j]
  arr[j] = temp
}
```

非递归写法：

```text
/**
 * 从i开始检查并保持大顶堆
 * @param arr 待排数组
 * @param i 检查的起始下标
 * @param size 堆大小
 */
function maxHeapify2(arr, i, size) {
  let left, right, maxlr = -1

  while (i < size) {
    left = 2 * i
    right = left + 1

    //无左右孩子节点
    if (left > size && right > size) {
      break
    }
    //只有左孩子节点
    if (left <= size && right > size) {
      maxlr = left
    }
    //只有右孩子节点
    if (right <= size && left > size) {
      maxlr = right
    }
    //同时有左右孩子节点
    if (left <= size && right <= size) {
      maxlr = arr[left] < arr[right] ? right : left
    }

    if (arr[i] < arr[maxlr]) {
      swap(arr, maxlr, i)
      i = maxlr
    }
  }
}

function swap(arr, i, j) {
  let temp = arr[i]
  arr[i] = arr[j]
  arr[j] = temp
}
```

### **创建大顶堆**

**创建大顶堆**（Build-Max-Heap）的作用是，将一个数组改造成一个大顶堆，会**自下而上**地调用 Max-Heapify 来改造数组。

```text
function buildMaxHeap(arr) {
  if (!Array.isArray(arr)) return []

  //将null插到数组第一个位置上
  arr.unshift(null)
  let lastParentIndex = Math.floor((arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    maxHeapify(arr, i, arr.length - 1)
  }
  arr.shift()
}
```

### **堆排序**

**堆排序**（Heap-Sort）是堆排序的接口算法，其先要调用创建大顶堆（Build-Max-Heap）将数组改造为大顶堆；
然后进入迭代，迭代中先将堆顶与堆底元素交换，并将堆长度缩短，继而重新调用大顶堆调整（Max-Heapify）保持大顶堆性质。

因为堆顶元素必然是堆中最大的元素，所以每一次操作之后，堆中存在的最大元素会被分离出堆，重复 n-1 次，数组排序完成。

```text
function heapSort(arr) {
  arr[0] !== null && arr.unshift(null)
  for (let j = arr.length - 1; j > 0; j--) {
    //将堆顶与堆底元素交换，分离出最大的元素
    swap(arr, 1, j) 
    //重新调整大顶堆
    maxHeapify(arr, 1, j - 1)
  }
  arr.shift()
}
```

我们来测试一下大顶堆的构建与排序：

```text
var arr = [5, 2, 8, 3, 1, 6, 9]
buildMaxHeap(arr)
console.log(arr)
heapSort(arr)
console.log(arr)

//[9,3,8,2,1,6,5]
//[1,2,3,5,6,8,9]
```

### **复杂度**

堆排序是一种选择排序，整体主要由构建初始堆+交换堆顶元素和末尾元素并重建堆两部分组成。

其中构建初始堆经推导复杂度为O(n)，在交换并重建堆的过程中，需交换n-1次，而重建堆的过程中，根据完全二叉树的性质，[log2(n-1),log2(n-2)…1]逐步递减，近似为nlogn。

所以堆排序时间复杂度一般认为就是`O(nlogn)`级。

### **添加元素**

将元素添加到数组末尾，然后进行大顶堆调整

```text
function maxHeapPush(arr = [], elem) {
  arr.push(elem)
  arr[0] !== null && arr.unshift(null)
  let lastParentIndex = Math.floor((arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    maxHeapify(arr, i, arr.length - 1)
  }
  arr.shift()
}
```

### **弹出元素**

每次只能弹出最值，即根节点，如果把根元素直接删除的话, 整个堆就毁了，
所以我们思考着使用内部的某一个元素先顶替根节点的位置，这个元素显而易见的是最后一个元素，因为最后一个元素的移动不会使得树的结构改变。
交换后，进行大顶堆调整。

```text
function maxHeapPop(arr = []) {
  arr[0] !== null && arr.unshift(null)
  swap(arr, 1, arr.length - 1)
  const top = arr.pop()

  let lastParentIndex = Math.floor((arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    maxHeapify(arr, i, arr.length - 1)
  }
  arr.shift()
  return top
}
```

测试一下：

```text
var arr = [5, 2, 8, 3, 1, 6, 9]
buildMaxHeap(arr)
console.log(arr)
maxHeapPush(arr, 10)
console.log(arr)
maxHeapPop(arr)
console.log(arr)

//[10, 9, 8, 3, 1, 6, 5, 2]
//[9, 3, 8, 2, 1, 6, 5]
```

封装一下，完整的代码如下：

```text
function Heap(type = 'max') {
  this.type = type
  this.arr = []
}

Heap.prototype.build = function() {
  this.arr.unshift(null)
  let lastParentIndex = Math.floor((this.arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    this.heapify(i, this.arr.length - 1)
  }
  this.arr.shift()
}

Heap.prototype.heapify = function(i, size) {
  let left = 2 * i
  let right = left + 1

  //左右孩子中较大或较小的一个
  let lr = -1

  //无左右孩子节点
  if (left > size && right > size) {
    return
  }
  //只有左孩子节点
  if (left <= size && right > size) {
    lr = left
  }
  //只有右孩子节点
  if (right <= size && left > size) {
    lr = right
  }
  //同时有左右孩子节点
  if (left <= size && right <= size) {
    lr = this.type === 'max' ? (this.arr[left] < this.arr[right] ? right : left) : (this.arr[left] > this.arr[right] ? right : left)
  }

  if ((this.type === 'max' && this.arr[i] < this.arr[lr]) || (this.type === 'min' && this.arr[i] > this.arr[lr])) {
    this.swap(i, lr)
    this.heapify(lr, size)
  }
}

Heap.prototype.sort = function() {
  this.arr[0] !== null && this.arr.unshift(null)
  for (let j = this.arr.length - 1; j > 0; j--) {
    this.swap(1, j)
    this.heapify(1, j - 1)
  }
  this.arr.shift()
}

Heap.prototype.add = function(elem) {
  this.arr.push(elem)
  this.arr[0] !== null && this.arr.unshift(null)
  let lastParentIndex = Math.floor((this.arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    this.heapify(i, this.arr.length - 1)
  }
  this.arr.shift()
}

Heap.prototype.pop = function() {
  this.arr[0] !== null && this.arr.unshift(null)
  this.swap(1, this.arr.length - 1)
  const top = this.arr.pop()

  let lastParentIndex = Math.floor((this.arr.length - 1) / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    this.heapify(i, this.arr.length - 1)
  }
  this.arr.shift()
  return top
}

Heap.prototype.swap = function(i, j) {
  let temp = this.arr[i]
  this.arr[i] = this.arr[j]
  this.arr[j] = temp
}

var heap = new Heap()
heap.add(5)
heap.add(2)
heap.add(8)
heap.add(3)
heap.add(1)
heap.add(6)
heap.add(9)
console.log(heap.arr)

//heap.build()
//console.log(heap.arr)

heap.add(10)
console.log(heap.arr)

heap.pop()
console.log(heap.arr)

heap.sort()
console.log(heap.arr)
```

### **数据流中的中位数**

如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。

如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。

步骤：

1. 维护一个大顶堆，一个小顶堆，数据总数：

- 小顶堆里的值全大于大顶堆里的；
- 2个堆个数的差值小于等于1

1. 当插入数字后数据总数为奇数时：使小顶堆个数比大顶堆多1；当插入数字后数据总数为偶数时，使大顶堆个数跟小顶堆个数一样。
2. 当总数字个数为奇数时，中位数就是小顶堆堆头；当总数字个数为偶数时，中位数数就是2个堆堆顶平均数。

```text
const maxHeap = new Heap('max');
const minHeap = new Heap('min');
let count = 0;
function Insert(num) {
  count++;
  if (count % 2 === 1) {
    maxHeap.add(num);
    minHeap.add(maxHeap.pop());
  } else {
    minHeap.add(num);
    maxHeap.add(minHeap.pop());
  }
}
function GetMedian() {
  if (count % 2 === 1) {
    return minHeap.value[0];
  } else {
    return (minHeap.value[0] + maxHeap.value[0]) / 2
  }
}
```

### **最小的k个数**

输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。

步骤：

1. 把前k个数构建一个大顶堆
2. 从第k个数开始，和大顶堆的最大值进行比较，若比最大值小，交换两个数的位置，重新构建大顶堆
3. 一次遍历之后大顶堆里的数就是整个数据里最小的k个数

```text
function getLeastNumbers(arr, k) {
  if (k > arr.length) {
    return []
  }
  arr.unshift(null)
  buildMaxHeap(arr, k + 1)

  for (let i = k + 1; i < arr.length; i++) {
    if (arr[i] < arr[1]) {
      [arr[i], arr[1]] = [arr[1], arr[i]]

      let lastParentIndex = Math.floor(k / 2)
      for (let j = lastParentIndex; j > 0; j--) {
        maxHeapify(arr, j, k)
      }
    }
  }
  return arr.slice(1, k + 1)
}

function buildMaxHeap(arr, size) {
  let lastParentIndex = Math.floor(size / 2)
  for (let i = lastParentIndex; i > 0; i--) {
    maxHeapify(arr, i, size)
  }
}

function maxHeapify(arr, i, size) {
  let left = 2 * i
  let right = left + 1

  //左右孩子中较大的一个
  let maxlr = -1

  //无左右孩子节点
  if (left > size && right > size) {
    return
  }
  //只有左孩子节点
  if (left <= size && right > size) {
    maxlr = left
  }
  //只有右孩子节点
  if (right <= size && left > size) {
    maxlr = right
  }
  //同时有左右孩子节点
  if (left <= size && right <= size) {
    maxlr = arr[left] < arr[right] ? right : left
  }

  if (arr[i] < arr[maxlr]) {
    [arr[i], arr[maxlr]] = [arr[maxlr], arr[i]]
    maxHeapify(arr, maxlr, size)
  }
}


var arr = [4, 5, 1, 6, 2, 7, 3, 8]
var result = getLeastNumbers(arr, 4)
console.log(result)
//[4,3,1,2]
```