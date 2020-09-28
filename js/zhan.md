### **概念**

- 栈是一种后进先出(LIFO，Last In First Out)的数据结构。**只用 pop 和 push** 完成增删的“数组”。
- 队列是一种先进先出（FIFO，First In First Out）的数据结构。**只用 push 和 shift** 完成增删的“数组”。

### **栈的实现**

```text
class Stack {
  constructor() {
    this.dataStore = []
  }
  // 模拟进栈的方法 
  push(element) {
    this.dataStore.push(element)
  }
  // 模拟出栈的方法，返回值是出栈的元素。
  pop() {
    return this.dataStore.pop()
  }
  // 返回栈顶元素
  peek() {
    return this.dataStore[dataStore.length - 1]
  }
  // 是否为空栈
  isEmpty() {
    return this.dataStore.length === 0
  }
  // 获取栈结构的长度。
  size() {
    return this.dataStore.length
  }
  // 清除栈结构内的所有元素。
  clear() {
    this.dataStore = []
  }
}
```

### **单向队列（顺序队列)**

```text
class Queue {
  constructor() {
    this.queue = []
  }
  enqueue(item) {
    this.queue.push(item)
  }
  dequeue() {
    return this.queue.shift()
  }
  head() {
    return this.queue[0]
  }
  isEmpty() {
    return this.size() === 0
  }
  size(){
    return this.queue.length
  }
}

var queue = new Queue();
console.log(queue.isEmpty()); // 输出 true
queue.enqueue('a');           // 添加元素 a
queue.enqueue('b');           // 添加元素 b
queue.enqueue('c');           // 添加元素 c
console.log(queue.queue)      // 输出 ['a','b','c']
console.log(queue.size());     // 输出 3
console.log(queue.isEmpty());  // 输出 false
queue.dequeue();               // 移除元素
queue.dequeue();
console.log(queue.queue)       // 输出 ['c']
```

### **优先队列**

优先队列是默认队列的变种，它的元素的添加和移除是基于优先级的。一个现实的例子就是医院的（急诊科）候诊室。医生会优先处理病情比较严重的患者。

实现一个优先队列，有两种选择：

- 设置优先级，然后在正确的位置添加元素；
- 用默认入列操作添加元素，按照优先级移除它们。

我们来实现第一种优先插入的情况：

```text
class QueueItem {
  constructor(item, priority) {
    this.item = item
    this.priority = priority
  }
}

class PriorityQueue {
  constructor() {
    this.queue = []
  }

  enqueue(item, priority) {
    const qItem = new QueueItem(item, priority)
    if (this.isEmpty()) {
      this.queue.push(qItem)
    } else {
      let added = false
      for (let i = 0; i < this.queue.length; i++) {
        if (priority < this.queue[i].priority) {
          this.queue.splice(i, 0, qItem) //在index:i的位置插入
          added = true
          break
        }
      }
      if (!added) {
        this.queue.push(qItem)
      }
    }
  }

  dequeue() {
    return this.queue.shift()
  }

  head() {
    return this.queue[0]
  }

  isEmpty() {
    return this.size() === 0
  }

  size() {
    return this.queue.length
  }

  print() {
    return this.queue.map(q => q.item)
  }
}

var queue = new PriorityQueue()
console.log(queue.isEmpty()) // 输出 true
queue.enqueue('a', 4)           // 添加元素 a
queue.enqueue('b', 2)           // 添加元素 b
queue.enqueue('c', 1)           // 添加元素 c
queue.enqueue('d', 1)           // 添加元素 d
console.log(queue.print())        // 输出 ['c','d','b','a']
```

### **循环队列**

循环队列是一种线性数据结构，其操作表现基于 FIFO（先进先出）原则并且队尾被连接在队首之后以形成一个循环。它也被称为“环形缓冲器”。

循环队列的一个好处是我们可以利用这个队列之前用过的空间。在一个普通队列里，一旦一个队列满了，我们就不能插入下一个元素，即使在队列前面仍有空间。
但是使用循环队列，我们能使用这些空间去存储新的值。

因为单链队列在出队操作的时候需要 O(n) 的时间复杂度，所以引入了循环队列。循环队列的出队操作平均是 O(1) 的时间复杂度。

典型的应用就是`击鼓传花`与`约瑟夫环`问题。

```text
class Queue {
  constructor() {
    this.queue = []
  }

  enqueue(item) {
    this.queue.push(item)
  }

  dequeue() {
    return this.queue.shift()
  }

  head() {
    return this.queue[0]
  }

  isEmpty() {
    return this.size() === 0
  }

  size() {
    return this.queue.length
  }
}

function flowerDrumTransfer(nodeList, counter) {
  const queue = new Queue()
  for (let i = 0; i < nodeList.length; i++) {
    queue.enqueue(nodeList[i])
  }

  let target = ''
  while (queue.size() > 1) {
    for (let j = 0; j < counter; j++) {
      queue.enqueue(queue.dequeue())
    }
    target = queue.dequeue() //该轮中被淘汰出局的
  }
  return queue.dequeue()
}

const arr = ['a', 'b', 'c', 'd', 'e']
const winner = flowerDrumTransfer(arr, 7)
console.log(winner)
```

### **栈与队列相互实现**

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

- 栈1:用于入队列存储
- 栈2:出队列时将栈1的数据依次出栈，并入栈到栈2中。栈2出栈即栈1的底部数据即队列要出的数据。

注意: 栈2为空才能补充栈1的数据，否则会打乱当前的顺序。

```text
const stack1 = []
const stack2 = []

function enqueue(n) {
  if (stack2.length === 0) {
    stack1.push(n)
  } else {
    while (stack2.length > 0) {
      stack1.push(stack2.pop())
    }
    stack1.push(n)
  }
}

function dequeue() {
  if (stack2.length === 0) {
    while (stack1.length > 0) {
      stack2.push(stack1.pop())
    }
  }
  return stack2.pop() || null
}
```

用两个队列实现一个栈。

```text
const queue1 = []
const queue2 = []

function push(x) {
  if (queue1.length === 0) {
    queue1.push(x)

    while (queue2.length) {
      queue1.push(queue2.shift())
    }
  } else if (queue2.length === 0) {
    queue2.push(x)

    while (queue1.length) {
      queue2.push(queue1.shift())
    }
  }
}

function pop() {
  if (queue1.length !== 0) {
    return queue1.shift()
  } else {
    return queue2.shift()
  }
}
```

### **栈中元素的最小值**

定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））

步骤：

1. 定义两个栈，一个栈用于存储数据，另一个栈用于存储每次数据进栈时栈的最小值.
2. 每次数据进栈时，将此数据和最小值栈的栈顶元素比较，将二者比较的较小值再次存入最小值栈.
3. 数据栈出栈，最小值栈也出栈。
4. 这样最小值栈的栈顶永远是当前栈的最小值

```text
const dataStack = []
const minStack = []

function push(node) {
  dataStack.push(node)

  if (node < min() || minStack.length === 0) {
    minStack.push(node)
  } else {
    minStack.push(min())
  }
}

function pop() {
  minStack.pop()
  return dataStack.pop()
}

function min() {
  const min = minStack[minStack.length - 1]
  return minStack.length && min
}
```

### **栈的压入弹出序列**

输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。

假设压入栈的所有数字均不相等。例如序列[1,2,3,4,5]是某栈的压入顺序，序列[4,5,3,2,1]是该压栈序列对应的一个弹出序列，但[4,3,5,1,2]就不可能是该压栈序列的弹出序列。
（注意：这两个序列的长度是相等的

步骤：

1. 借助一个辅助栈来模拟压入、弹出的过程，设置一个索引`index`，标志弹出序列，初始位置是弹出序列的第一位。
2. 按压入序列依次入栈，当辅助栈的栈顶元素与弹出序列的第一个元素相同时，辅助栈出栈，弹出序列的标志索引后移，即`index+1`，然后辅助栈继续入栈。
3. 所有数据都入栈后，如果出栈顺序正确，辅助栈应该为空。

```text
function IsPopOrder(pushV, popV) {
  if (!pushV || !popV || !pushV.length || !popV.length) {
    return
  }
  const stack = []
  let index = 0
  for (let i = 0; i < pushV.length; i++) {
    stack.push(pushV[i])
    while (stack.length && stack[stack.length - 1] == popV[index]) {
      stack.pop()
      index++
    }
  }
  return stack.length === 0
}

const result = IsPopOrder([1, 2, 3, 4, 5], [4, 5, 3, 2, 1])
console.log(result) //true
```