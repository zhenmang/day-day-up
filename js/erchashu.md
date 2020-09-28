## 二叉树

- 它可以没有根结点，作为一棵空树存在
- 如果它不是空树，那么必须由根结点、左子树和右子树组成，且左右子树都是二叉树。

![event-loop](/imgs/erchashu.png)

它的结构分为三块：

- 数据域
- 左侧子结点（左子树根结点）的引用
- 右侧子结点（右子树根结点）的引用

```
// 二叉树结点的构造函数
function TreeNode(val) {
    this.val = val;
    this.left = this.right = null;
}
```

### **基本结构**

```
function TreeNode(val) {
  this.val = val
  this.left = this.right = null
}

TreeNode.prototype = {
  show: function() {
    console.log(this.val)
  },
}

function BST() {
  this.root = null
}

BST.prototype = {
  insert: function(val) {
    var node = new TreeNode(val)
    if (!this.root) {
      this.root = node
      return
    }

    var current = this.root
    var parent = null
    while (current) {
      parent = current
      if (val < current.val) {
        current = current.left
        if (!current) {
          parent.left = node
          return
        }
      } else {
        current = current.right
        if (!current) {
          parent.right = node
          return
        }
      }
    }
  },
  preOrder: function(node) {
    if (node) {
      node.show()
      this.preOrder(node.left)
      this.preOrder(node.right)
    }
  },
  inOrder: function(node) {
    if (node) {
      this.inOrder(node.left)
      node.show()
      this.inOrder(node.right)
    }
  },
  postOrder: function(node) {
    if (node) {
      this.postOrder(node.left)
      this.postOrder(node.right)
      node.show()
    }
  },
  getMin: function() {
    var current = this.root
    while (current) {
      if (!current.left) {
        current.show()
        return
      }
      current = current.left
    }
  },
  getMax: function() {
    var current = this.root
    while (current) {
      if (!current.right) {
        current.show()
        return
      }
      current = current.right
    }
  },
  getDeep: function(node, deep) {
    deep = deep || 0
    if (node == null) {
      return deep
    }
    deep++;
    var dleft = this.getDeep(node.left, deep)
    var dright = this.getDeep(node.right, deep)
    return Math.max(dleft, dright)
  },
  getNode: function (data, node) {
      if (node) {
          if (data === node.data) {
              return node;
          } else if (data < node.data) {
              return this.getNode(data,node.left);
          } else {
              return this.getNode(data,node.right);
          }
      } else {
          return null;
      }
  }
}
```

验证一下

![event-loop](/imgs/erchashuyanzheng.png)

```
var t = new BST();
t.insert(3);
t.insert(8);
t.insert(1);
t.insert(2);
t.insert(5);
t.insert(7);
t.insert(6);
t.insert(0);
console.log(t);

console.log('中序:');
t.inOrder(t.root);//0 1 2 3 5 6 7 8 

console.log('先序:');
t.preOrder(t.root);//3 1 0 2 8 5 7 6 

console.log('后序:');
t.postOrder(t.root);//0 2 1 6 7 5 8 3

console.log(t.getMin(), t.getMax());
console.log(t.getDeep(t.root, 0));
console.log(t.getNode(5,t.root));
```

### **二分查找**

```
function binarySearch(target, arr, start, end) {
  if (start > end) {
    return -1
  }
  var mid = Math.floor((start + end) / 2)
  if (target === arr[mid]) {
    return mid
  } else if (target < arr[mid]) {
    return binarySearch(target, arr, start, mid - 1)
  } else {
    return binarySearch(target, arr, mid + 1, end)
  }
}
var arr = [0, 0, 1, 2, 3, 5, 4, 6, 7, 8]
console.log(binarySearch(1, arr, 0, arr.length-1)); 
```

### **中序遍历**

```
function inOrder(root, arr = []) {
  if(node){
     inOrder(root.left, arr)
     arr.push(root.val)
     inOrder(root.right, arr)
  }
}
```

非递归写法

- 取跟节点为目标节点，开始遍历
- 1.左孩子入栈 -> 直至左孩子为空的节点
- 2.节点出栈 -> 访问该节点
- 3.以右孩子为目标节点，再依次执行1、2、3

```
function inOrderV2(node) {
    var current = node
    var stack = []
    //出栈
    while (current || stack.length > 0) {
      //入栈
      while (current) {
        stack.push(current)
        current = current.left
      }
      current = stack.pop()
      current.show()
      current = current.right
    }
}
```

### **前序遍历**

```text
function preOrder(node) {
    if (node) {
      node.show()
      this.preOrder(node.left)
      this.preOrder(node.right)
    }
}
```

非递归写法

- 取跟节点为目标节点，开始遍历
- 1.访问目标节点
- 2.左孩子入栈 -> 直至左孩子为空的节点
- 3.节点出栈，以右孩子为目标节点，再依次执行1、2、3

```text
function preOrderV2(node) {
    var current = node
    var stack = []
    //出栈
    while (current || stack.length > 0) {
      //入栈
      while (current) {
        current.show()
        stack.push(current)
        current = current.left
      }
      current = stack.pop()
      current = current.right
    }
}
```

### **后序遍历**

```text
function postOrder(node) {
    if (node) {
      this.postOrder(node.left)
      this.postOrder(node.right)
      node.show()
    }
}
```

非递归写法

- 取跟节点为目标节点，开始遍历
- 1.左孩子入栈 -> 直至左孩子为空的节点
- 2.栈顶节点的右节点为空或右节点被访问过 -> 节点出栈并访问他，将节点标记为已访问
- 3.栈顶节点的右节点不为空且未被访问，以右孩子为目标节点，再依次执行1、2、3

```text
function postOrderV2(node) {
    var current = node
    var stack = []
    var last = null

    //出栈
    while (current || stack.length > 0) {
      //入栈
      while (current) {
        stack.push(current)
        current = current.left
      }

      current = stack[stack.length - 1]
      if (!current.right || current.right == last) {
        current = stack.pop()
        current.show()
        last = current
        current = null //继续弹栈
      } else {
        current = current.right
      }
    }
}
```

### **广度遍历**

广度遍历是从二叉树的根结点开始，自上而下逐层遍历；在同一层中，按照从左到右的顺序对结点逐一访问。

```text
function levelOrder(node, que = [node]) {
    node = que.shift()
    if (node) {
      node.show()
      if (node.left) {
        que.push(node.left)
      }
      if (node.right) {
        que.push(node.right)
      }
      this.levelOrder(node, que)
    }
}
```

非递归写法：

```text
function levelOrderV2(node) {
  if(node) {
     var que = []
     que.push(node)
     while(que.length > 0){
        node = que.shift()
        node.show()
        if(node.left){
           que.push(node.left)
        }
        if(node.right){
           que.push(node.right)
        }
     }
  }
}
```

### **二叉树重建**

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。

例如输入前序遍历序列`{1,2,4,7,3,5,6,8}`和中序遍历序列`{4,7,2,1,5,3,8,6}`，则重建二叉树并返回。

步骤：

1. 前序遍历找到根结点root
2. 找到root在中序遍历的位置 -> 左子树的长度和右子树的长度
3. 截取左子树的中序遍历、右子树的中序遍历
4. 截取左子树的前序遍历、右子树的前序遍历
5. 递归重建二叉树

```text
function reConstruct(pre, vin) {
  if(pre.length === 0){
     return null
  }
  if(pre.length === 1){
     return new TreeNode(pre[0])
  }
  var index = vin.indexOf(pre[0]) 
  var vinLeft =  vin.slice(0, index)
  var vinRight =  vin.slice(index+1)
  var preLeft = pre.slice(1, index+1)
  var preRight = pre.slice(index+1)
  const node = new TreeNode(value);
  node.left = reConstruct(preLeft, vinLeft);
  node.right = reConstruct(preRight, vinRight);
  return  node
}
```

### **求二叉树遍历**

给定一棵二叉树的前序遍历和中序遍历，求其后序遍历。

输入描述:

两个字符串，其长度n均小于等于26。 第一行为前序遍历，第二行为中序遍历。 二叉树中的结点名称以大写字母表示：A，B，C….最多26个结点。

输出描述:

输入样例可能有多组，对于每组测试样例， 输出一行，为后序遍历的字符串。

步骤：

1. 前序遍历找到根结点root
2. 找到root在中序遍历的位置 -> 左子树的长度和右子树的长度
3. 截取左子树的中序遍历、右子树的中序遍历
4. 截取左子树的前序遍历、右子树的前序遍历
5. 递归拼接后序遍历

```text
function getHRD(pre, vin, totalLength) {
  if (!pre) {
    return '';
  }
  if (pre.length === 1) {
    return pre;
  }
  const head = pre[0];
  const splitIndex = vin.indexOf(head);
  const vinLeft = vin.substring(0, splitIndex);
  const vinRight = vin.substring(splitIndex + 1);
  const preLeft = pre.substring(1, splitIndex + 1);
  const preRight = pre.substring(splitIndex + 1);

  const result = getHRD(preLeft, vinLeft) + getHRD(preRight, vinRight) + head
  if(result.length === totalLength){
    console.log(result)
  }
  return result;
}

getHRD('ABC','BAC',3)
getHRD('FDXEAG','XDEFAG',6)

> 输出：
BCA
XEDGAF
```

### **对称二叉树(镜像)**

请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

步骤：

1. 两个根结点相等
2. 左子树的右节点和右子树的左节点相同。
3. 右子树的左节点和左子树的右节点相同。

```text
function isSymmetrical(a, b) {
  if (!a || !b || a !== b) {
    return false
  }
  if (isSymmetrical(a.right) !== isSymmetrical(b.left)) {
    return false
  }
  if (isSymmetrical(a.left) !== isSymmetrical(b.right)) {
    return false
  }
  return true
}
```

操作给定的二叉树，将其变换为源二叉树的镜像。

```text
function mirror(root) {
  if(root){
    const left = root.left
    const right =  root.right
    root.left = right
    root.right = left
    mirror(root.left)
    mirror(root.right)
  }
}
```

### **二叉搜索树的第k个节点**

给定一棵**二叉搜索树**，请找出其中的第k小的节点。

二叉搜索树概念：

- 若任意节点的左⼦子树不不空，则左⼦子树上所有结点的值均⼩小于它的 根结点的值;
- 若任意节点的右⼦子树不不空，则右⼦子树上所有结点的值均⼤大于它的 根结点的值;
- 任意节点的左、右⼦子树也分别为⼆二叉查找树。

例如， （5，3，7，2，4，6，8） 中，按节点数值大小顺序第三小节点的值为4

步骤：

二叉搜索树的中序遍历就是从小到大顺序输出的。

```text
//递归版
function searchK(root, k) {
  const arr = []
  loopThrough(root, arr)
  if (k > 0 && k <= arr.length) {
    return arr[k - 1]
  }
}

function loopThrough(node, arr) {
  if (node) {
    loopThrough(node.left, arr)
    arr.push(node)
    loopThrough(node.right, arr)
  }
}

//非递归版
function searchKv2(root, k) {
  var arr = []
  var stack = []
  var current = node

  while(current || stack.length > 0){

    while (current){
      stack.push(current)
      current = current.left
    }

    current = stack.pop()
    arr.push(current)
    current = current.right
  }

  if (k > 0 && k <= arr.length) {
    return arr[k - 1]
  }
}
```

### **二叉搜索树的后续遍历**

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。
如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。

步骤：

1. 后序遍历逐步输出到一个数组中
2. 输出过程中，如果某个索引，在原数组与输出数组中对应的值不等，则停止输出，输出No
3. 否则一直输出所有后序遍历的结果，所有值全部都相同，则输出Yes

```text
function postOrderv2(root, inputArr) {
  var outArr = []
  var stack = []
  var current = node
  var last = null
  var result = 'Yes'

  while(current || stack.length > 0){

    while (current){
      stack.push(current)
      current = current.left
    }

    current = stack[stack.length-1]
    if(!current.right || current.right == last){
      current = stack.pop()
      if(inputArr[outArr.length] === current){
        outArr.push(current)
      }else{
        result = 'No'
        break
      }
      last = current
      current = null
    }else{
      current = current.right
    }

    return result
  }
}
```

### **二叉树最大(小)深度**

给定一个二叉树，找出其最大深度。(采用递归+分治)

**二叉树的深度**为根节点到最远叶子节点的最长路径上的节点数。

一棵二叉树的最大深度 = 左子树与右子树最大深度的最大值 + 1

```text
function maxDepth(root) {
   return !root?0:Math.max(maxDepth(root.left),maxDepth(root.right)) + 1
}
```

**二叉树的深度**是从根节点到最近叶子节点的最短路径上的节点数量。

步骤：

1. 左树为空：右子树最小深度 + 1
2. 右树为空：左子树最小深度 + 1
3. 左右子树都不为空：左子树和右子树最小深度的最小值 + 1

```text
function minDepth(root) {
   if(!root){
      return 0
   }
   if(!root.left){
      return 1 + minDepth(root.right)
   }
   if(!root.right){
      return 1 + minDepth(root.left)
   }
   return Math.min(minDepth(root.left),minDepth(root.right)) + 1
}
```

### **平衡二叉树**

**平衡二叉树**：每个子树的深度之差不超过1

输入一棵二叉树，判断该二叉树是否是平衡二叉树。

步骤：

1. 递归遍历左右子树，比较深度，若差大于1，则返回-1，表示当前子树不平衡
2. 若左右子树有一个不平衡的，或左右子树差大于1，则整棵树不平衡
3. 若整棵树平衡，则返回当前树的深度（左右子树深度的最大值+1）

```text
function isBalanced(root) {
   return balanced(root) != -1
}

function balanced(node) {
  if(!node){
    return 0
  }
  const left = balanced(node.left)
  const right = balanced(node.right)
  if(left == -1 || right == -1 || Math.abs(left - right) > 1){
    return -1
  }
  return Math.max(left, right) + 1
}
```

### **二叉树中和为某个值的路径**

输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。
路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。

步骤：

- 设定一个结果数组result来存储所有符合条件的**路径**
- 设定一个栈stack来存储当前路径中的节点
- 设定一个和sum来标识当前路径之和

1. 从根结点开始深度优先遍历，每经过一个节点，将节点入栈
2. 到达**叶子节点**，且当前路径之和等于给定目标值，则找到一个可行的解决方案，将其加入结果数组
3. 遍历到二叉树的某个节点时有2个可能的选项，选择前往左子树或右子树
4. 若存在左子树，继续向左子树递归
5. 若存在右子树，继续向右子树递归
6. 若上述条件均不满足，或已经遍历过，将当前节点出栈，向上回溯

```text
function findPath(root, num) {
  var result = []
  if(root){
     findPathCore(root, num, result)
  }
  return result
}

function findPathCore(node, num, stack, sum, result) {
   stack.push(node.val)
   sum += node.val
   if(!node.left && !node.right && sum === num){
     result.push(stack.slice(0))
   }
   if(node.left){
     findPathCore(node.left, num, stack, sum, result)
   }
   if(node.right){
     findPathCore(node.right, num, stack, sum, result)
   }
   stack.pop()
}
```

### **二叉树的序列化与反序列化**

- 若一颗二叉树是不完全的，我们至少需要两个遍历才能将它重建（像题目重建二叉树一样）
- 但是这种方式仍然有一定的局限性，比如二叉树中不能出现重复节点。
- 如果二叉树是一颗完全二叉树，我们只需要知道前序遍历即可将它重建。
- 因此在序列化时二叉树时，可以将空节点使用特殊符号存储起来，这样就可以模拟一棵完全二叉树的前序遍历
- 在重建二叉树时，当遇到特殊符号当空节点进行处理

```text
function Serialize(root, arr=[]) {
  if(!root){
    arr.push('#')
  }else{
    arr.push(root.val)
    Serialize(root.left, arr)
    Serialize(root.right, arr)
  }
  return arr.join(',')
}

function Deserialize(str) {
  if(!str){
    return null
  }
  return deserialize(s.split(','))
}

function deserialize(arr) {
  let node;
  const current = arr.shift()
  if(current !==  '#'){
    node = {val: current}
    node.left = deserialize(arr)
    node.right = deserialize(arr)
  }
  return node
}
```

### **二叉树的下一个节点**

给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

- 右节点不为空 - 取右节点的最左侧节点
- 右节点为空 - 如果节点是父亲节的左节点 取父节点
- 右节点为空 - 如果节点是父亲节的右节点 父节点已经被遍历过，再往上层寻找…
- 左节点一定在当前节点之前被遍历过

![event-loop](/imgs/nextnode.png)

```text
/*function TreeLinkNode(x){
    this.val = x;
    this.left = null;
    this.right = null;
    this.next = null;
}*/
function getNext(node, node) {
  if(!node){
    return null
  }
  if(node.right){
    node = node.right
    while (node.left){
      node = node.left
    }
    return node
  }else{
    while (node){
      if(!node.next){
        return null
      }else if(node = node.next.left){
        return node.next
      }
      node = node.next
    }
    return node
  }
}
```

### **树的子结构**

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）

步骤：

1. 首先找到A树中和B树根节点相同的节点
2. 从此节点开始，递归AB树比较是否有不同节点

```text
function hasSubtree(r1,r2) {
  let result = false
  if(r1 && r2){
    if(r1.val === r2.val){
      result = compare(r1,r2)
    }
    if(!result){
      result = hasSubtree(r1.right, r2)
    }
    if (!result) {
      result = hasSubtree(r1.left, r2);
    }
  }
  return result
}

function compare(r1,r2) {
  if(r2 === null){
    return true
  }
  if(r1 === null){
    return false
  }
  if(r1.val !== r2.val){
    return false
  }
  return compare(r1.right, r2.right) &&  compare(r1.left, r2.left)
}
```